# Database Selection: PostgreSQL vs DynamoDB vs MongoDB

## Core Decision Framework

Choosing the right database is not about which technology is "best" — it is about which one fits your workload. Evaluate along five axes:

| Axis | Key Question |
|------|-------------|
| **Data Model** | Is your data relational with complex joins, key-value/document, or schema-flexible? |
| **Access Patterns** | Are queries predictable and narrow, or ad-hoc and analytical? |
| **Consistency** | Do you need strong ACID transactions, or is eventual consistency acceptable? |
| **Scale** | Are you scaling reads, writes, or both? Do you need single-digit-ms latency at millions of RPS? |
| **Operational Overhead** | Do you have a DBA team, or do you need a fully managed, near-zero-ops solution? |

> **Rule of thumb:** Start with PostgreSQL unless you have a specific, demonstrable reason not to. It is the safest default for most applications.

---

## PostgreSQL

### When to Use

- Your data is **relational** — normalized tables with foreign keys, joins, and referential integrity.
- You need **complex queries**: aggregations, window functions, CTEs, full-text search.
- **Strong consistency** (ACID) is non-negotiable — financial transactions, inventory, user accounts.
- You want a **single database** that handles OLTP and light OLAP without a separate warehouse.
- Your scale is in the range of **tens of thousands of QPS** (which covers the vast majority of applications).

### Real-World Signals That Point to PostgreSQL

- "We need to join 3-4 tables to answer most business questions."
- "Our schema is well-defined and changes infrequently."
- "We run nightly analytical reports against the same database."
- "We need row-level security or multi-tenant isolation."
- "We want PostGIS for geospatial queries."

### Strengths

- Rich SQL dialect, mature optimizer, excellent extension ecosystem (PostGIS, pg_trgm, pgvector).
- MVCC-based concurrency — readers never block writers.
- JSONB columns give you document-like flexibility within a relational model.
- Battle-tested replication (streaming, logical) and tooling.
- Massive community and operational knowledge base.

### Limitations

- Vertical scaling has a ceiling; horizontal write-scaling requires sharding (Citus) or application-level partitioning.
- Connection-per-process model can be a bottleneck at high concurrency (mitigated by PgBouncer).
- Not designed for single-digit-millisecond latency at millions of requests per second.

---

## DynamoDB

### When to Use

- Your access patterns are **known in advance** and can be modeled as key-value or key-partition lookups.
- You need **single-digit-millisecond latency at any scale** — millions of RPS with consistent performance.
- You want **zero operational overhead** — no patching, no capacity planning, no replication setup.
- Your workload is **write-heavy** and globally distributed (Global Tables).

### Real-World Signals That Point to DynamoDB

- "We can enumerate every query the application will ever make."
- "Our traffic is spiky and unpredictable; we need auto-scaling with no downtime."
- "We are already deep in the AWS ecosystem (Lambda, API Gateway, EventBridge)."
- "We need sub-10ms P99 latency for a user-session or shopping-cart service."

### The Predictable Access Pattern Requirement

DynamoDB forces you to **design your data model around your queries**, not around your entities. This means:

- You must know your access patterns before you design your table.
- Single-table design crams multiple entity types into one table using composite sort keys (e.g., `PK=USER#123`, `SK=ORDER#2024-01-15`).
- Adding a new access pattern after launch can require a **full table migration** or a new GSI (max 20 per table).

This is not a limitation — it is a **trade-off**. You get predictable performance at any scale, but you pay with upfront design effort and reduced query flexibility.

### The DynamoDB Trap

Teams fall into the DynamoDB trap when they:

1. **Choose DynamoDB for its brand/hype** without validating that access patterns are truly predictable.
2. **Discover new query requirements** mid-project (e.g., "now we need to search by email AND date range").
3. **Bolt on GSIs** to patch the gap, adding cost and complexity.
4. **End up with a system that is harder to operate** than PostgreSQL would have been — the opposite of the original goal.

> **Guard rail:** If your PM says "we might need ad-hoc reporting later," DynamoDB is probably not your primary store.

### Strengths

- Consistent single-digit-ms latency regardless of table size.
- Fully managed: automatic scaling, built-in backup, point-in-time recovery.
- Global Tables for multi-region active-active replication.
- Seamless integration with AWS serverless stack.
- DynamoDB Streams for event-driven architectures (CDC).

### Limitations

- 400 KB item size limit.
- No server-side joins — all joining happens in application code.
- Query flexibility is fundamentally limited by partition key and sort key design.
- Scans are expensive and slow; they defeat the purpose of DynamoDB.
- Eventual consistency by default (strongly consistent reads cost 2x and only work against the leader).

---

## MongoDB

### When to Use

- Your data is **document-shaped** — nested objects, arrays, polymorphic records that vary across documents.
- Your schema is **evolving rapidly** — early-stage product, frequent schema migrations would be painful.
- You need **flexible querying** on nested fields without predefined access patterns.
- Your use case involves **content management, catalogs, user profiles, or event logging** where each record can look different.

### Real-World Signals That Point to MongoDB

- "Each product in our catalog has different attributes depending on category."
- "We are iterating on the data model weekly; we cannot afford downtime for ALTER TABLE."
- "We need to query deeply nested fields (e.g., `items.variants.color`)."
- "We want rich secondary indexes without the single-table-design complexity of DynamoDB."

### Flexible Schema Use Cases

MongoDB's schema-on-read model shines when:

- **Polymorphic data**: An `events` collection where click events, purchase events, and error events have different shapes.
- **Embedded documents**: A `user` document containing an array of `addresses`, each with optional fields.
- **Rapid prototyping**: Schema changes are just code changes — no migration scripts, no downtime.
- **Semi-structured data**: Ingesting JSON from third-party APIs where the schema is not under your control.

### Strengths

- Flexible document model with rich query language (aggregation pipeline, text search, geospatial).
- Horizontal scaling via native sharding.
- Multi-document ACID transactions (since 4.0).
- Change Streams for event-driven patterns (similar to DynamoDB Streams).
- Atlas (managed service) reduces operational burden significantly.

### Limitations

- Transactions are supported but carry a performance cost; not designed for high-contention transactional workloads.
- No enforced referential integrity — dangling references are your problem.
- Joins (`$lookup`) are expensive compared to SQL joins; denormalization is the default pattern.
- WiredTiger storage engine can exhibit write amplification under heavy load.

---

## Side-by-Side Comparison

| Dimension | PostgreSQL | DynamoDB | MongoDB |
|-----------|-----------|----------|---------|
| **Data Model** | Relational (tables, rows, joins) | Key-value / wide-column | Document (JSON/BSON) |
| **Schema** | Schema-on-write (strict) | Schema-less (app-enforced) | Schema-on-read (flexible) |
| **Query Language** | SQL | PartiQL / GetItem / Query API | MQL (aggregation pipeline) |
| **Joins** | Native, optimized | None (application-side) | `$lookup` (limited) |
| **Consistency** | Strong (ACID, serializable) | Eventual by default, strong reads available | Strong within replica set |
| **Scaling Model** | Vertical + read replicas | Horizontal (auto-partitioned) | Horizontal (sharded) |
| **Write Throughput** | Thousands of TPS | Millions of TPS | Tens of thousands of TPS |
| **Latency at Scale** | Low-ms, degrades under extreme load | Single-digit-ms, constant | Low-ms, depends on working set |
| **Operational Burden** | Medium (tuning, vacuuming, backups) | Very low (fully managed) | Low–Medium (Atlas) to High (self-hosted) |
| **Best For** | General-purpose, relational, analytical | High-scale, predictable access, serverless | Flexible schema, rapid iteration, documents |

---

## Polyglot Persistence

Most production systems at scale do not use a single database. The **polyglot persistence** pattern means choosing the right database for each bounded context:

```
┌─────────────────────────────────────────────────────┐
│                    Application                      │
├──────────┬──────────────┬──────────┬────────────────┤
│  Users   │   Sessions   │ Catalog  │  Analytics     │
│  (PG)    │  (DynamoDB)  │ (Mongo)  │  (ClickHouse)  │
└──────────┴──────────────┴──────────┴────────────────┘
```

**When to apply polyglot persistence:**

- Different services have fundamentally different data models and access patterns.
- A single database cannot satisfy both latency and query-flexibility requirements.
- Teams are autonomous and can own their data stores independently.

**Costs:**

- Increased operational complexity (monitoring, backups, expertise across multiple systems).
- Cross-store consistency requires coordination (sagas, outbox pattern, CDC).
- Data duplication and synchronization overhead.

> **Interview tip:** Mention polyglot persistence when a design has clearly distinct workloads. Show that you understand both the benefit (right tool for each job) and the cost (operational complexity, cross-store consistency).

---

## Concurrency Support: Optimistic vs Pessimistic Locking

| Database | Pessimistic Locking | Optimistic Locking |
|----------|--------------------|--------------------|
| **PostgreSQL** | Native — `SELECT ... FOR UPDATE`, `FOR SHARE`, advisory locks | Native — use a `version` column and `UPDATE ... WHERE version = ?` |
| **DynamoDB** | Not supported — no row-level locks | Native — **condition expressions** (`attribute_not_exists`, `version = :expected`) on write operations |
| **MongoDB** | Implicit via transactions — acquiring a transaction on a document effectively locks it for the duration | Manual — implement a `version` field, read-then-write with version check; no built-in primitive |

### PostgreSQL: Both Approaches Native

PostgreSQL supports both locking strategies out of the box:

- **Pessimistic:** `SELECT ... FOR UPDATE` acquires a row-level exclusive lock. Other transactions block until the lock is released. Ideal for high-contention scenarios (e.g., seat reservation, inventory decrement).
- **Optimistic:** Add a `version` or `updated_at` column. Read the current version, then `UPDATE ... SET version = version + 1 WHERE id = ? AND version = ?`. If zero rows are affected, another transaction won. Ideal for low-contention, read-heavy workloads.

### DynamoDB: Optimistic Only via Condition Expressions

DynamoDB has no concept of locks or transactions that block other writers. Instead:

- Every write (`PutItem`, `UpdateItem`, `DeleteItem`) can include a **condition expression**.
- Pattern: Store a `version` attribute. On update, include `ConditionExpression: "version = :expected"` and increment the version. If the condition fails, the write is rejected with `ConditionalCheckFailedException`.
- This is the **only concurrency control mechanism** in DynamoDB — there is no way to "hold" a lock.

### MongoDB: Implicit Pessimistic + Manual Optimistic

- **Pessimistic (implicit):** Multi-document transactions (since 4.0) acquire intent locks on documents. While a transaction is open, other writers to the same document will block or abort. This gives you pessimistic-like behavior, but it is a side effect of the transaction system, not an explicit locking API.
- **Optimistic (manual):** MongoDB has no built-in optimistic concurrency primitive. You implement it yourself: add a `version` field, use `findOneAndUpdate` with a `version` filter. If the update matches zero documents, another writer won.

---

## Key Takeaways

1. **PostgreSQL is the safe default.** Unless you have a specific, validated reason to choose something else, start here. Most applications never outgrow it.

2. **DynamoDB is a specialist.** It excels at massive scale with predictable access patterns. It punishes you if your access patterns change. Always validate your query patterns before committing.

3. **MongoDB fills the flexible-schema gap.** When your data is genuinely document-shaped and your schema is evolving, MongoDB gives you queryability that DynamoDB does not and flexibility that PostgreSQL does not.

4. **Polyglot persistence is the real-world answer.** Large systems use multiple databases. Show that you can pick the right one for each context and handle the cross-store coordination costs.

5. **Concurrency models differ fundamentally.** PostgreSQL gives you both pessimistic and optimistic locking natively. DynamoDB offers optimistic only (condition expressions). MongoDB gives you implicit pessimistic via transactions and manual optimistic via version fields.

---

## Interview Talking Points

- **"Why not just use PostgreSQL for everything?"** — Because at extreme scale (millions of RPS, single-digit-ms P99), you hit vertical scaling limits. DynamoDB removes that ceiling but restricts query flexibility. It is a trade-off, not an upgrade.

- **"When would you reach for DynamoDB?"** — When I can enumerate all access patterns upfront, need auto-scaling with zero ops, and the data fits a key-value or key-partition model. Classic examples: session stores, shopping carts, leaderboards, IoT telemetry.

- **"Why MongoDB over PostgreSQL's JSONB?"** — PostgreSQL JSONB is great for occasional semi-structured columns, but MongoDB is purpose-built for document workloads: richer indexing on nested fields, native sharding, and a query language designed for documents. If most of your data is documents, MongoDB is more ergonomic.

- **"How do you handle consistency across multiple databases?"** — Sagas for distributed workflows, the transactional outbox pattern for reliable event publishing, and CDC (Change Data Capture) for synchronizing data between stores. Avoid distributed transactions (2PC) unless absolutely necessary — they are slow and fragile.

- **"How do you handle concurrent writes?"** — It depends on the database and contention level. PostgreSQL gives me both `SELECT FOR UPDATE` (pessimistic, for high contention) and version-column checks (optimistic, for low contention). DynamoDB only supports optimistic concurrency through condition expressions. MongoDB provides implicit pessimistic locking through transactions and manual optimistic locking through application-level version checks.
