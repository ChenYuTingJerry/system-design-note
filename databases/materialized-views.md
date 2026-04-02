# Materialized Views: PostgreSQL, Redshift, MongoDB, DynamoDB

## Core Concept

### The Single Idea: Pre-computation

A **materialized view** is a query result that is computed once and stored physically, so subsequent reads return the pre-computed result instead of re-executing the query. The core idea is **trading storage and freshness for read speed**.

```
Regular view:         Query → parse → plan → execute → scan tables → join → aggregate → return
Materialized view:    Query → read pre-computed result from disk → return
```

### Regular View vs Materialized View

| Aspect | Regular View | Materialized View |
|--------|-------------|-------------------|
| Storage | None — it is an alias for a query | Physical — stores the result set on disk |
| Read speed | Same as running the underlying query | Fast — reads from stored data |
| Freshness | Always current | Stale until refreshed |
| Can be indexed | No (it is just a query rewrite) | Yes — you can create indexes on the MV |
| Write overhead | None | Refresh cost (full or incremental) |

### How It Gets Fast

The performance gain comes from three layers stacked together:

```
1. Pre-computation     →  expensive joins/aggregations run once, not per request
2. Physical storage    →  result lives on disk (or in memory), not recomputed
3. Indexes on the MV   →  you can index the materialized result for sub-ms lookups

Combined: a 30-second analytical query becomes a 2ms indexed read.
```

---

## How It Works Across Databases

### PostgreSQL: CREATE MATERIALIZED VIEW

PostgreSQL has first-class materialized view support with explicit refresh control.

#### Creating a Materialized View

```sql
-- Pre-aggregate daily revenue per product
CREATE MATERIALIZED VIEW mv_daily_revenue AS
SELECT
  product_id,
  DATE_TRUNC('day', order_time) AS order_date,
  SUM(amount) AS total_revenue,
  COUNT(*) AS order_count
FROM orders
GROUP BY product_id, DATE_TRUNC('day', order_time);

-- Add indexes on the MV for fast lookups
CREATE INDEX idx_mv_daily_revenue_product ON mv_daily_revenue (product_id);
CREATE INDEX idx_mv_daily_revenue_date ON mv_daily_revenue (order_date);
```

#### Refreshing

```sql
-- Full refresh: locks the MV, replaces all data
REFRESH MATERIALIZED VIEW mv_daily_revenue;

-- Concurrent refresh: no lock on reads, but requires a UNIQUE index
CREATE UNIQUE INDEX idx_mv_daily_revenue_pk
  ON mv_daily_revenue (product_id, order_date);

REFRESH MATERIALIZED VIEW CONCURRENTLY mv_daily_revenue;
```

| Refresh Mode | Behavior | Downside |
|-------------|----------|----------|
| `REFRESH MATERIALIZED VIEW` | Locks the MV — readers block during refresh | Downtime for reads |
| `REFRESH ... CONCURRENTLY` | Swaps data in-place — readers see old data until done | Requires a `UNIQUE` index; slower refresh (diff-based) |

**Key caveat:** PostgreSQL does not support automatic or incremental refresh. You must trigger `REFRESH` manually or via a scheduler (cron, pg_cron).

```sql
-- Using pg_cron to refresh every 10 minutes
SELECT cron.schedule('refresh-mv-daily-revenue', '*/10 * * * *',
  'REFRESH MATERIALIZED VIEW CONCURRENTLY mv_daily_revenue');
```

---

### Redshift: AUTO REFRESH and Query Rewriting

Amazon Redshift treats materialized views as a first-class optimization primitive with features PostgreSQL lacks.

```sql
-- Create with auto refresh enabled
CREATE MATERIALIZED VIEW mv_daily_revenue
AUTO REFRESH YES
AS
SELECT
  product_id,
  DATE_TRUNC('day', order_time) AS order_date,
  SUM(amount) AS total_revenue,
  COUNT(*) AS order_count
FROM orders
GROUP BY product_id, DATE_TRUNC('day', order_time);
```

#### Redshift-Specific Features

| Feature | Description |
|---------|------------|
| **AUTO REFRESH** | Redshift automatically refreshes the MV when base tables change. No cron needed. |
| **Incremental refresh** | Only processes new/changed rows instead of recomputing the full result. Requires supported query patterns (single-table aggregations, no outer joins). |
| **Query rewriting** | The optimizer automatically rewrites queries to read from the MV even if the query does not reference the MV directly. The user writes `SELECT ... FROM orders GROUP BY ...`, and Redshift transparently serves it from the MV. |

**Query rewriting example:**

```sql
-- User writes this query
SELECT product_id, SUM(amount)
FROM orders
WHERE order_time >= '2025-01-01'
GROUP BY product_id;

-- Redshift optimizer rewrites it to read from mv_daily_revenue
-- This is transparent — the user does not know the MV exists
```

**Incremental refresh constraints:** The MV must use only `SUM`, `COUNT`, `MIN`, `MAX`, or `AVG` over a single base table. Joins, subqueries, and `HAVING` clauses disable incremental refresh and force full recomputation.

---

### MongoDB: Simulated via $out / $merge

MongoDB does not have a native materialized view. Instead, you **simulate** one by running an aggregation pipeline and writing the result to a separate collection using `$out` or `$merge`.

```javascript
// Compute daily revenue and write to a "materialized" collection
db.orders.aggregate([
  {
    $group: {
      _id: {
        product_id: "$product_id",
        order_date: { $dateTrunc: { date: "$order_time", unit: "day" } }
      },
      total_revenue: { $sum: "$amount" },
      order_count: { $sum: 1 }
    }
  },
  {
    $project: {
      _id: 0,
      product_id: "$_id.product_id",
      order_date: "$_id.order_date",
      total_revenue: 1,
      order_count: 1
    }
  },
  // $out replaces the entire collection (atomic swap)
  { $out: "mv_daily_revenue" }
]);

// $merge is more flexible — upsert, merge, or replace matched docs
db.orders.aggregate([
  // ... same pipeline ...
  {
    $merge: {
      into: "mv_daily_revenue",
      on: ["product_id", "order_date"],    // match key
      whenMatched: "replace",
      whenNotMatched: "insert"
    }
  }
]);
```

**After writing, index the output collection:**

```javascript
db.mv_daily_revenue.createIndex({ product_id: 1 });
db.mv_daily_revenue.createIndex({ order_date: 1 });
```

| Stage | Behavior |
|-------|----------|
| `$out` | Atomically replaces the target collection. Simple but always a full refresh. |
| `$merge` | Upserts into the target collection. Enables incremental-like updates by matching on a key. |

**Refresh is your responsibility** — schedule the aggregation via cron, a background job, or an application-level scheduler.

---

### DynamoDB: Simulated via Streams + Lambda + Summary Table

DynamoDB has no materialized view or aggregation capability. You simulate one by combining **DynamoDB Streams**, **Lambda**, and a **separate summary table**.

```
┌──────────┐    Stream     ┌──────────┐    Write     ┌──────────────────┐
│  orders  │ ──────────▶  │  Lambda  │ ──────────▶  │ daily_revenue    │
│  (base)  │   (CDC)      │ (compute)│   (upsert)   │ (summary table)  │
└──────────┘              └──────────┘              └──────────────────┘
```

```javascript
// Lambda function triggered by DynamoDB Streams
exports.handler = async (event) => {
  const updates = {};

  for (const record of event.Records) {
    if (record.eventName === 'INSERT' || record.eventName === 'MODIFY') {
      const newImage = record.dynamodb.NewImage;
      const productId = newImage.product_id.S;
      const orderDate = newImage.order_time.S.substring(0, 10); // YYYY-MM-DD
      const amount = parseFloat(newImage.amount.N);

      const key = `${productId}#${orderDate}`;
      if (!updates[key]) {
        updates[key] = { productId, orderDate, totalRevenue: 0, orderCount: 0 };
      }
      updates[key].totalRevenue += amount;
      updates[key].orderCount += 1;
    }
  }

  // Atomic upsert into summary table
  const { DynamoDBClient } = require('@aws-sdk/client-dynamodb');
  const { DynamoDBDocumentClient, UpdateCommand } = require('@aws-sdk/lib-dynamodb');
  const client = DynamoDBDocumentClient.from(new DynamoDBClient());

  for (const update of Object.values(updates)) {
    await client.send(new UpdateCommand({
      TableName: 'daily_revenue',
      Key: { product_id: update.productId, order_date: update.orderDate },
      UpdateExpression: 'ADD total_revenue :rev, order_count :cnt',
      ExpressionAttributeValues: {
        ':rev': update.totalRevenue,
        ':cnt': update.orderCount
      }
    }));
  }
};
```

**Key design points:**

- DynamoDB Streams provides CDC (Change Data Capture) — Lambda receives every write event.
- The summary table uses `ADD` (atomic increment) to handle concurrent Lambda invocations safely.
- This is **event-driven refresh** — the summary updates within seconds of a base table write.
- The pattern is powerful but requires managing Lambda concurrency, error handling, and dead-letter queues.

---

## Scenarios: When to Use Materialized Views

### Reporting and Dashboards

Dashboard queries aggregate millions of rows. Pre-compute once, serve instantly.

```sql
-- PostgreSQL: executive dashboard MV
CREATE MATERIALIZED VIEW mv_executive_dashboard AS
SELECT
  region,
  DATE_TRUNC('month', order_time) AS month,
  SUM(amount) AS revenue,
  COUNT(DISTINCT customer_id) AS unique_customers,
  AVG(amount) AS avg_order_value
FROM orders
GROUP BY region, DATE_TRUNC('month', order_time);
```

### Pre-aggregating Time-Series Data

Roll up granular events into hourly/daily summaries. Queries on the summary are orders of magnitude faster than scanning raw events.

```sql
-- PostgreSQL: roll up per-minute metrics into hourly
CREATE MATERIALIZED VIEW mv_hourly_metrics AS
SELECT
  service_name,
  DATE_TRUNC('hour', recorded_at) AS hour,
  AVG(cpu_usage) AS avg_cpu,
  MAX(cpu_usage) AS peak_cpu,
  PERCENTILE_CONT(0.95) WITHIN GROUP (ORDER BY latency_ms) AS p95_latency
FROM metrics_raw
GROUP BY service_name, DATE_TRUNC('hour', recorded_at);
```

### Flattening Complex JOINs

Pre-join normalized tables into a denormalized view for fast reads.

```sql
-- PostgreSQL: flatten orders + customers + products
CREATE MATERIALIZED VIEW mv_order_details AS
SELECT
  o.id AS order_id,
  o.order_time,
  o.amount,
  c.name AS customer_name,
  c.email AS customer_email,
  p.name AS product_name,
  p.category
FROM orders o
JOIN customers c ON o.customer_id = c.id
JOIN products p ON o.product_id = p.id;

CREATE INDEX idx_mv_order_details_customer ON mv_order_details (customer_email);
CREATE INDEX idx_mv_order_details_category ON mv_order_details (category, order_time);
```

### Search and Filtering Optimization

Create a pre-flattened, pre-indexed collection optimized for search.

```javascript
// MongoDB: flatten product + reviews + inventory into a search-optimized collection
db.products.aggregate([
  {
    $lookup: {
      from: "reviews",
      localField: "_id",
      foreignField: "product_id",
      as: "reviews"
    }
  },
  {
    $lookup: {
      from: "inventory",
      localField: "_id",
      foreignField: "product_id",
      as: "inventory"
    }
  },
  {
    $project: {
      name: 1,
      category: 1,
      price: 1,
      avg_rating: { $avg: "$reviews.rating" },
      review_count: { $size: "$reviews" },
      in_stock: { $gt: [{ $sum: "$inventory.quantity" }, 0] }
    }
  },
  { $out: "mv_product_search" }
]);

db.mv_product_search.createIndex({ category: 1, avg_rating: -1 });
db.mv_product_search.createIndex({ price: 1 });
```

### Billing Reports (Redshift)

Redshift's auto-refresh and query rewriting make MVs ideal for billing pipelines where analysts write ad-hoc queries against large fact tables.

```sql
-- Redshift: billing summary MV with auto refresh
CREATE MATERIALIZED VIEW mv_monthly_billing
AUTO REFRESH YES
AS
SELECT
  account_id,
  DATE_TRUNC('month', usage_time) AS billing_month,
  service_type,
  SUM(cost_usd) AS total_cost,
  SUM(usage_units) AS total_usage
FROM usage_events
GROUP BY account_id, DATE_TRUNC('month', usage_time), service_type;

-- Analysts query usage_events directly; Redshift rewrites to use the MV
```

---

## Pros and Cons

| Aspect | Pros | Cons |
|--------|------|------|
| **Read performance** | Dramatic speedup — pre-computed results with indexes | No benefit for write-heavy workloads |
| **Query simplicity** | Complex joins/aggregations hidden behind a simple table scan | Adds a new object to manage and monitor |
| **Storage** | Trades cheap disk for expensive compute time | Duplicate data — the MV stores a copy of derived data |
| **Freshness** | Acceptable for many use cases (dashboards, reports) | Data is stale between refreshes — not suitable for real-time needs without CDC |
| **Indexability** | Can create indexes on the MV for further optimization | Indexes on the MV also need storage and write maintenance |
| **Operational cost** | Reduces load on base tables during peak read times | Refresh jobs add background compute load; must be monitored |

---

## Patterns

### Scheduled Refresh

The simplest pattern. A cron job or scheduler triggers a full or concurrent refresh at a fixed interval.

```sql
-- PostgreSQL: refresh every 15 minutes via pg_cron
SELECT cron.schedule('refresh-mv', '*/15 * * * *',
  'REFRESH MATERIALIZED VIEW CONCURRENTLY mv_daily_revenue');
```

**When to use:** Dashboards and reports where minutes of staleness are acceptable. Low complexity, easy to reason about.

### Event-Driven Refresh (CDC)

Base table changes trigger an immediate refresh of the materialized view. Uses Change Data Capture to detect writes.

```
Write to base table → CDC event → trigger refresh or incremental update
```

**When to use:** Near-real-time requirements where scheduled refresh is too stale. The DynamoDB Streams + Lambda pattern above is an example.

### Layered Materialized Views

Build MVs on top of other MVs to create a hierarchy of pre-computations.

```sql
-- Layer 1: hourly aggregation
CREATE MATERIALIZED VIEW mv_hourly_sales AS
SELECT
  product_id,
  DATE_TRUNC('hour', order_time) AS hour,
  SUM(amount) AS revenue
FROM orders
GROUP BY product_id, DATE_TRUNC('hour', order_time);

-- Layer 2: daily aggregation built on hourly
CREATE MATERIALIZED VIEW mv_daily_sales AS
SELECT
  product_id,
  DATE_TRUNC('day', hour) AS day,
  SUM(revenue) AS revenue
FROM mv_hourly_sales
GROUP BY product_id, DATE_TRUNC('day', hour);

-- Refresh order matters: hourly first, then daily
REFRESH MATERIALIZED VIEW CONCURRENTLY mv_hourly_sales;
REFRESH MATERIALIZED VIEW CONCURRENTLY mv_daily_sales;
```

**Caution:** Refresh order matters — child MVs depend on parent MVs being current. A refresh orchestrator must enforce the correct order.

### Partial Materialized View

Only materialize a subset of the data — recent records, a specific tenant, or a hot partition.

```sql
-- Only materialize the last 30 days of data
CREATE MATERIALIZED VIEW mv_recent_orders AS
SELECT
  product_id,
  DATE_TRUNC('day', order_time) AS order_date,
  SUM(amount) AS revenue,
  COUNT(*) AS order_count
FROM orders
WHERE order_time >= CURRENT_DATE - INTERVAL '30 days'
GROUP BY product_id, DATE_TRUNC('day', order_time);
```

**When to use:** When the full dataset is too large to materialize, but the hot working set is a small fraction. Reduces refresh time and storage.

---

## CDC and Event-Driven Integration

### CDC Pipeline Architecture

A production CDC pipeline for maintaining materialized views at scale:

```
┌────────────┐    CDC     ┌─────────┐  Stream  ┌─────────┐  Write  ┌──────────────────┐
│ PostgreSQL │ ────────▶ │Debezium │ ──────▶ │  Kafka  │ ──────▶ │     Flink        │
│ (source)   │  WAL-based │(capture)│         │ (buffer)│         │ (transform +     │
└────────────┘           └─────────┘         └─────────┘         │  aggregate)      │
                                                                  └────────┬─────────┘
                                                                           │ Write
                                                                           ▼
                                                                  ┌──────────────────┐
                                                                  │ Materialized     │
                                                                  │ Store (PG/Redis/ │
                                                                  │ Elasticsearch)   │
                                                                  └──────────────────┘
```

- **Debezium** reads the PostgreSQL WAL (or MongoDB oplog, or MySQL binlog) and emits change events.
- **Kafka** buffers and distributes change events with ordering guarantees per partition.
- **Flink** consumes the stream, applies windowed aggregations or joins, and writes the result to the materialized store.

### Scheduled Refresh vs CDC-Driven Refresh

| Dimension | Scheduled Refresh | CDC-Driven Refresh |
|-----------|------------------|--------------------|
| **Staleness** | Minutes to hours | Seconds to low minutes |
| **Complexity** | Low — cron + `REFRESH` | High — Debezium + Kafka + Flink (or Streams + Lambda) |
| **Compute pattern** | Batch — full recomputation on schedule | Streaming — incremental updates per event |
| **Base table load** | Re-reads all source data on each refresh | Zero — reads from CDC log, not the base table |
| **Failure mode** | Missed refresh → stale data | Consumer lag → stale data; poison pill → pipeline stall |
| **Best for** | Dashboards, nightly reports, low-change data | Real-time analytics, live leaderboards, event-driven systems |
| **Ordering guarantees** | N/A — full snapshot each time | Must handle out-of-order events (Kafka partition ordering) |
| **Infrastructure cost** | Low — just the database | Higher — Kafka cluster, Flink jobs, monitoring |

---

## Redis as a Materialized View

Redis is not a traditional database with materialized view support, but its data structures make it an excellent **in-memory materialized view store** when sub-millisecond reads are required.

### Hash for Pre-computed Records

Store a pre-computed aggregate as a Redis Hash — each field is a metric.

```javascript
const Redis = require('ioredis');
const redis = new Redis();

// Write: pre-compute and store product daily revenue
async function materializeProductRevenue(productId, date, revenue, orderCount) {
  const key = `mv:product_revenue:${productId}:${date}`;
  await redis.hset(key, {
    total_revenue: revenue.toString(),
    order_count: orderCount.toString(),
    updated_at: new Date().toISOString()
  });
  await redis.expire(key, 86400 * 7); // TTL: 7 days
}

// Read: sub-millisecond lookup
async function getProductRevenue(productId, date) {
  const key = `mv:product_revenue:${productId}:${date}`;
  return await redis.hgetall(key);
}
```

### Sorted Set for Rankings

Sorted Sets are purpose-built for leaderboards and top-N queries.

```javascript
// Write: update streamer score in real-time
async function updateStreamerRanking(streamerId, giftValue) {
  await redis.zincrby('mv:top_streamers', giftValue, streamerId);
}

// Read: top 20 streamers in < 1ms
async function getTopStreamers(limit = 20) {
  return await redis.zrevrange('mv:top_streamers', 0, limit - 1, 'WITHSCORES');
}
```

### TTL-Based Expiring Materialized View

Use TTL to automatically expire stale materialized data, forcing a re-computation on the next read (lazy refresh).

```javascript
async function getOrComputeDashboard(tenantId) {
  const key = `mv:dashboard:${tenantId}`;
  const cached = await redis.get(key);
  if (cached) return JSON.parse(cached);

  // Cache miss: compute from source database
  const result = await computeDashboardFromDB(tenantId);
  await redis.set(key, JSON.stringify(result), 'EX', 300); // TTL: 5 minutes
  return result;
}
```

### Redis MV vs PostgreSQL MV

| Dimension | Redis MV | PostgreSQL MV |
|-----------|----------|---------------|
| **Storage** | In-memory (RAM cost) | On-disk (cheap storage) |
| **Read latency** | Sub-millisecond | Low milliseconds (with index) |
| **Data size** | Limited by RAM — hot/small datasets | Unlimited — full analytical results |
| **Refresh** | Application-driven (write-through, TTL, CDC) | `REFRESH MATERIALIZED VIEW` (manual or cron) |
| **Indexing** | Built into data structure (Hash, Sorted Set) | Standard B-tree/GIN indexes |
| **Persistence** | Volatile by default (RDB/AOF optional) | Fully durable |
| **Query capability** | Key lookup, range on sorted set | Full SQL |
| **Best for** | Real-time leaderboards, session data, hot aggregates | Reports, dashboards, complex analytical queries |

---

## Real-World Scenarios

### Live Streamer Recommendations (17LIVE-Style)

**Problem:** Show the top 20 highest-earning streamers in the last hour, updated in real-time, with < 5ms read latency.

**Architecture:**

```
┌──────────────┐   Event   ┌─────────┐  Stream  ┌─────────────────┐  Write  ┌───────────────┐
│ Gift Service │ ────────▶ │  Kafka  │ ──────▶ │  Flink Job      │ ──────▶ │ Redis Sorted  │
│ (produces    │           │ (topic: │         │ (rolling 1-hour │         │ Set           │
│  gift events)│           │  gifts) │         │  window SUM)    │         │ top_streamers │
└──────────────┘           └─────────┘         └─────────────────┘         └───────┬───────┘
                                                                                    │
                                                                             Read (< 5ms)
                                                                                    │
                                                                           ┌────────▼────────┐
                                                                           │  API Server      │
                                                                           │  GET /top-streamers
                                                                           └─────────────────┘
```

**Flink job (conceptual):**

```sql
-- Flink SQL: rolling 1-hour window of gift values per streamer
INSERT INTO redis_top_streamers
SELECT
  streamer_id,
  SUM(gift_value) AS total_gifts
FROM gifts_stream
GROUP BY
  streamer_id,
  HOP(event_time, INTERVAL '1' MINUTE, INTERVAL '1' HOUR);
```

**Redis read path:**

```javascript
// API endpoint: GET /top-streamers
async function getTopStreamers(req, res) {
  // ZREVRANGE is O(log(N) + M) where M is the number of elements returned
  const top20 = await redis.zrevrange('mv:top_streamers:current_hour', 0, 19, 'WITHSCORES');

  const result = [];
  for (let i = 0; i < top20.length; i += 2) {
    result.push({
      streamer_id: top20[i],
      total_gifts: parseFloat(top20[i + 1])
    });
  }

  res.json(result); // < 5ms total
}
```

**Why this works:** Flink handles the expensive rolling-window aggregation in a streaming fashion. Redis Sorted Set gives O(log N) writes and O(log N + M) reads. The API server does zero computation — it just reads pre-computed results.

---

### Follower/Follow Count in Social Networks

**Problem:** Display follower/following counts instantly. The `follows` table has billions of rows — `COUNT(*)` is too slow.

**Architecture:**

```
┌──────────────┐   CDC    ┌─────────┐  Stream  ┌─────────┐  INCR/DECR  ┌───────────────┐
│   follows    │ ──────▶ │Debezium │ ──────▶ │  Kafka  │ ──────────▶ │ Redis         │
│   (PG table) │  WAL    │         │         │         │             │ follower_count│
└──────────────┘         └─────────┘         └─────────┘             └───────┬───────┘
                                                                              │
                                                                       Read (< 1ms)
                                                                              │
                                                                     ┌────────▼────────┐
                                                                     │  Profile API     │
                                                                     │  GET /user/:id   │
                                                                     └──────────────────┘
```

**CDC consumer (Kafka consumer):**

```javascript
const { Kafka } = require('kafkajs');
const Redis = require('ioredis');
const redis = new Redis();

const kafka = new Kafka({ brokers: ['kafka:9092'] });
const consumer = kafka.consumer({ groupId: 'follower-count-materializer' });

await consumer.subscribe({ topic: 'postgres.public.follows' });

await consumer.run({
  eachMessage: async ({ message }) => {
    const event = JSON.parse(message.value.toString());
    const followeeId = event.after?.followee_id || event.before?.followee_id;
    const followerId = event.after?.follower_id || event.before?.follower_id;

    if (event.op === 'c') {
      // INSERT: new follow — increment both counts atomically
      await redis.incr(`mv:follower_count:${followeeId}`);
      await redis.incr(`mv:following_count:${followerId}`);
    } else if (event.op === 'd') {
      // DELETE: unfollow — decrement both counts atomically
      await redis.decr(`mv:follower_count:${followeeId}`);
      await redis.decr(`mv:following_count:${followerId}`);
    }
  }
});
```

**The Race Condition Problem:**

If two users follow the same person at the same instant, and you use read-then-write (GET → compute → SET), you can lose an increment:

```
Thread A: GET follower_count:user123 → 1000
Thread B: GET follower_count:user123 → 1000
Thread A: SET follower_count:user123 → 1001
Thread B: SET follower_count:user123 → 1001  ← should be 1002
```

**The Atomic INCR Solution:**

Redis `INCR` and `DECR` are **atomic** — they are single commands executed by Redis's single-threaded event loop. No read-modify-write race is possible.

```javascript
// INCR is atomic — even with 10,000 concurrent follows, every one is counted
await redis.incr(`mv:follower_count:${followeeId}`);
// Equivalent to: read → increment → write, but as one indivisible operation
```

**Periodic Reconciliation for Drift Correction:**

Over time, edge cases cause drift: failed CDC events, duplicate processing, consumer restarts. A reconciliation job periodically corrects the counts from the source of truth.

```javascript
// Reconciliation job: runs nightly or hourly
async function reconcileFollowerCounts(pgPool) {
  // Source of truth: PostgreSQL
  const { rows } = await pgPool.query(`
    SELECT followee_id, COUNT(*) AS follower_count
    FROM follows
    GROUP BY followee_id
  `);

  for (const row of rows) {
    const redisCount = await redis.get(`mv:follower_count:${row.followee_id}`);
    const pgCount = parseInt(row.follower_count);

    if (parseInt(redisCount) !== pgCount) {
      console.log(`Drift detected for user ${row.followee_id}: redis=${redisCount}, pg=${pgCount}`);
      await redis.set(`mv:follower_count:${row.followee_id}`, pgCount);
    }
  }
}
```

**Why this pattern matters:** It combines the correctness of PostgreSQL (source of truth) with the speed of Redis (materialized view). CDC keeps them in near-real-time sync, INCR eliminates race conditions, and reconciliation handles the inevitable drift.

---

## Support Matrix

| Feature | PostgreSQL | Redshift | MongoDB | DynamoDB |
|---------|-----------|----------|---------|----------|
| **Native MV** | Yes — `CREATE MATERIALIZED VIEW` | Yes — `CREATE MATERIALIZED VIEW` | No — simulated via `$out`/`$merge` | No — simulated via Streams + Lambda |
| **Auto refresh** | No — manual or pg_cron | Yes — `AUTO REFRESH YES` | No — application-scheduled | N/A — event-driven by design |
| **Incremental refresh** | No — always full recomputation | Yes — for single-table aggregations | Partial — `$merge` can upsert changed docs | N/A — Lambda processes individual events |
| **Query rewriting** | No — must query the MV directly | Yes — optimizer rewrites transparently | No | No |
| **Concurrent refresh** | Yes — `REFRESH ... CONCURRENTLY` | N/A — auto refresh handles this | N/A — `$out` does atomic swap | N/A |
| **Workaround** | N/A (native) | N/A (native) | Aggregation pipeline + `$out`/`$merge` + cron | DynamoDB Streams + Lambda + summary table |

---

## The One-Variable Framework

When designing a materialized view, three decisions define the architecture:

### Where: Disk vs Memory vs Separate Table

| Option | Example | Trade-off |
|--------|---------|-----------|
| **Same database, on disk** | PostgreSQL `MATERIALIZED VIEW` | Simplest; shares resources with base tables |
| **In memory** | Redis Hash / Sorted Set | Sub-ms reads; limited by RAM; volatile without persistence |
| **Separate table/database** | DynamoDB summary table, Elasticsearch index | Independent scaling; more infrastructure to manage |

### When: Scheduled vs Event-Driven vs On Write

| Option | Example | Trade-off |
|--------|---------|-----------|
| **Scheduled** | pg_cron `REFRESH` every 15 min | Simple; predictable load; minutes of staleness |
| **Event-driven** | CDC → Kafka → consumer → update MV | Near-real-time; complex infrastructure |
| **On write** | Application updates MV in same transaction | Zero staleness; write amplification; tight coupling |

### How Stale: Seconds vs Minutes vs Hours

| Staleness | Use Case | Pattern |
|-----------|----------|---------|
| **Seconds** | Live leaderboard, follower counts | CDC → streaming processor → Redis |
| **Minutes** | Operational dashboards, search indexes | Scheduled refresh every 5–15 min |
| **Hours** | Nightly reports, billing summaries, analytics | Scheduled refresh on cron (hourly/daily) |

> **Interview tip:** When discussing materialized views, anchor on these three variables. They frame every design decision and show you understand the trade-off space, not just individual tools.

---

## Key Takeaways

1. **Materialized views are pre-computation made persistent.** The core idea is simple: run an expensive query once, store the result, and serve reads from the stored copy. Everything else is about managing when and how to refresh that copy.

2. **PostgreSQL gives you the most control.** Native `CREATE MATERIALIZED VIEW`, `CONCURRENTLY` refresh, and the ability to index the MV. But you must manage refresh scheduling yourself.

3. **Redshift is the most automated.** Auto refresh, incremental refresh, and transparent query rewriting make MVs a first-class optimization tool. If you are in the Redshift ecosystem, use them aggressively.

4. **MongoDB and DynamoDB require you to build it yourself.** MongoDB uses `$out`/`$merge` aggregation pipelines. DynamoDB uses Streams + Lambda + summary tables. Both work, but the operational burden is on you.

5. **Redis is the best in-memory materialized view store.** When you need sub-millisecond reads on pre-computed data, Redis data structures (Hash, Sorted Set) are purpose-built for this.

6. **CDC pipelines are the production-grade refresh mechanism.** Debezium → Kafka → Flink → materialized store is the standard architecture for near-real-time materialized views at scale.

7. **Atomic operations solve the concurrency problem.** Redis `INCR`/`DECR` and DynamoDB `ADD` eliminate read-modify-write races. Use them for counters and aggregates in your materialized views.

8. **Always plan for drift.** No matter how good your CDC pipeline is, edge cases will cause the materialized view to diverge from the source of truth. Build periodic reconciliation jobs to correct drift.

---

## Interview Talking Points

- **"What is a materialized view?"** — It is a query result that is pre-computed and physically stored. Instead of executing an expensive join or aggregation on every read, you compute it once and serve reads from the stored result. The trade-off is freshness: the data is stale until you refresh it.

- **"When would you use a materialized view vs a cache?"** — A materialized view is a structured, query-aware pre-computation — it understands the schema and can be indexed. A cache is a generic key-value store of serialized results. Use a materialized view when you need to query the pre-computed data (filter, sort, join against it). Use a cache when you just need to store and retrieve an exact response blob.

- **"How do you keep a materialized view fresh?"** — Three approaches: scheduled refresh (cron-based, simplest), event-driven refresh (CDC pipeline, near-real-time), or synchronous refresh on write (in the same transaction, zero staleness but high write cost). The choice depends on how stale the data can be.

- **"How would you build a real-time leaderboard?"** — Gift events flow into Kafka, Flink computes a rolling-window aggregation, and writes to a Redis Sorted Set. The API reads the top N from the Sorted Set in sub-millisecond time. This is a materialized view pattern — Flink is the refresh mechanism, Redis is the materialized store.

- **"What about race conditions in follower counts?"** — Use Redis `INCR`/`DECR` which are atomic single-threaded operations, not read-modify-write. Feed follow/unfollow events through CDC into a Kafka consumer that calls `INCR`/`DECR`. Run a periodic reconciliation job against the source of truth to correct any drift from edge cases like duplicate processing or failed events.

- **"How does this differ across databases?"** — PostgreSQL and Redshift have native materialized views. Redshift adds auto refresh, incremental refresh, and query rewriting. MongoDB simulates MVs via `$out`/`$merge` aggregation pipelines. DynamoDB simulates them via Streams + Lambda + summary tables. The pattern is the same — pre-compute and store — but the mechanism varies by database.
