# Covering Indexes and Partial Indexes: PostgreSQL, DynamoDB, MongoDB

## Covering Indexes

### Core Idea

A **covering index** is an index that contains all the columns (or attributes) a query needs, so the database can answer the query entirely from the index without touching the main table/collection. This eliminates the most expensive part of a query — the random I/O to fetch rows from the heap or document store.

```
Regular index:       Index → find pointers → fetch rows from table → return data
Covering index:      Index → return data directly (no table access)
```

The performance gain is dramatic: a covering index turns what would be thousands of random disk reads into a sequential index scan.

---

### PostgreSQL: Index-Only Scans and INCLUDE

PostgreSQL calls a query answered entirely from an index an **Index-Only Scan**. The planner chooses this when every column in `SELECT`, `WHERE`, `JOIN`, and `ORDER BY` is present in the index.

#### Basic Covering Index

```sql
-- Query: SELECT email, created_at FROM users WHERE status = 'active' ORDER BY created_at;

-- Covering index: all three columns are in the index
CREATE INDEX idx_users_covering
  ON users (status, created_at, email);
```

The planner sees that `status`, `created_at`, and `email` are all in the index and uses an Index-Only Scan — no heap fetch required.

#### INCLUDE vs Key Columns

PostgreSQL 11 introduced `INCLUDE` to separate **search columns** (used for filtering/ordering) from **payload columns** (only needed in output):

```sql
CREATE INDEX idx_users_covering
  ON users (status, created_at)
  INCLUDE (email);
```

| Aspect | Key columns (`status`, `created_at`) | INCLUDE columns (`email`) |
|--------|--------------------------------------|---------------------------|
| Used for filtering/ordering | Yes | No |
| Maintained in B-tree sort order | Yes | No — stored as unsorted payload in leaf pages |
| Can satisfy `WHERE` / `ORDER BY` | Yes | No |
| Adds to index internal node size | Yes — increases tree depth | No — only in leaf pages |
| Supports uniqueness constraints | Yes | No |

**Why INCLUDE matters:**

- Key columns bloat the internal nodes of the B-tree, making the tree deeper and reducing cache efficiency.
- INCLUDE columns are only stored in leaf pages, keeping the tree shallow.
- Use `INCLUDE` for columns you only need in the `SELECT` list — they ride along cheaply.

```sql
-- Unique constraint + covering: enforce uniqueness on (tenant_id, email)
-- while also covering display_name for Index-Only Scans
CREATE UNIQUE INDEX idx_users_unique_email
  ON users (tenant_id, email)
  INCLUDE (display_name);
```

#### The Visibility Map / VACUUM Caveat

PostgreSQL's MVCC means multiple versions of a row can exist in the heap. An Index-Only Scan must verify that the row version visible to the current transaction actually exists — but it does not want to visit the heap to check.

The solution is the **visibility map**: a bitmap with one bit per heap page. If a page's bit is set, all tuples on that page are visible to all transactions, and the index can skip the heap visit.

```
Index-Only Scan decision per tuple:
  1. Find tuple in index
  2. Check visibility map for that heap page
     → Bit SET:   return data from index (fast path)
     → Bit UNSET: must fetch heap page to check visibility (slow path, defeats purpose)
```

**The caveat:** The visibility map is updated by `VACUUM`. If `VACUUM` has not run recently (or cannot keep up with writes), many pages will have their bits unset, and the "Index-Only Scan" degrades into a regular Index Scan with heap fetches.

**Practical implications:**

- High-write tables need aggressive autovacuum tuning for Index-Only Scans to be effective.
- Monitor `Heap Fetches` in `EXPLAIN (ANALYZE, BUFFERS)` — a high number means the visibility map is stale.
- `pg_stat_user_tables.n_dead_tup` tells you how many dead tuples are waiting for VACUUM.

```sql
EXPLAIN (ANALYZE, BUFFERS) SELECT email, created_at FROM users WHERE status = 'active';
-- Look for:
--   Index Only Scan ...
--     Heap Fetches: 12      ← low = visibility map is healthy
--     Heap Fetches: 48,301  ← high = VACUUM is behind, index-only scan is not effective
```

---

### DynamoDB: GSI Projections

DynamoDB's equivalent of a covering index is a **Global Secondary Index (GSI) with projected attributes**.

When you create a GSI, you choose which attributes to **project** into the index:

| Projection Type | What is Stored | Equivalent To |
|----------------|---------------|---------------|
| `KEYS_ONLY` | Table key + GSI key only | Non-covering index |
| `INCLUDE` | Table key + GSI key + specified attributes | Covering index for those attributes |
| `ALL` | All attributes from the base table | Fully covering (but 1:1 storage cost) |

```json
{
  "IndexName": "GSI-email-created",
  "KeySchema": [
    { "AttributeName": "status", "KeyType": "HASH" },
    { "AttributeName": "created_at", "KeyType": "RANGE" }
  ],
  "Projection": {
    "ProjectionType": "INCLUDE",
    "NonKeyAttributes": ["email", "display_name"]
  }
}
```

**Key differences from PostgreSQL:**

- If a query requests an attribute **not projected** into the GSI, DynamoDB will **not** automatically fetch from the base table — it simply will not return that attribute. There is no transparent "fallback to heap."
- GSIs are **eventually consistent** (no strongly consistent reads on GSIs).
- Each GSI is a full copy of the projected data, consuming its own read/write capacity and storage.
- Maximum 20 GSIs per table.

**Cost implication:** `ALL` projection makes every query "covered" but doubles your storage cost and write throughput consumption. Choose `INCLUDE` with only the attributes your queries actually need.

---

### MongoDB: Covered Queries (IXSCAN with No FETCH)

MongoDB calls a query that is answered entirely from the index a **covered query**. In `explain()` output, this appears as an `IXSCAN` stage with **no `FETCH` stage** following it.

```javascript
// Create a compound index
db.users.createIndex({ status: 1, created_at: 1, email: 1 });

// Covered query: all fields in the index, _id excluded from projection
db.users.find(
  { status: "active" },
  { _id: 0, email: 1, created_at: 1 }
).sort({ created_at: 1 });
```

**Critical rule: you must exclude `_id` from the projection** (`_id: 0`), unless `_id` is part of the index. By default, MongoDB returns `_id`, which forces a FETCH to the collection.

```javascript
// Verify with explain()
db.users.find({ status: "active" }, { _id: 0, email: 1, created_at: 1 })
  .sort({ created_at: 1 })
  .explain("executionStats");

// Look for:
//   winningPlan.stage: "IXSCAN"           ← no FETCH stage above it
//   totalDocsExamined: 0                  ← confirms no collection access
```

**MongoDB vs PostgreSQL covering indexes:**

- MongoDB has no `INCLUDE` equivalent — all covered fields must be key columns in the compound index, which means they affect index sort order and size.
- No visibility map issue — MongoDB does not have MVCC in the same way; WiredTiger uses a different concurrency model.
- Covered queries work with sharded collections, but only if the shard key is part of the query or index.

---

## Partial Indexes

### Core Idea

A **partial index** indexes only a subset of rows/documents that match a predicate. Rows that do not match the predicate are not in the index at all.

```
Full index:     Indexes ALL rows     → large, slower to maintain
Partial index:  Indexes SOME rows    → smaller, faster to maintain, faster to scan
```

**Benefits:**

- **Smaller index size** — less disk, more of the index fits in memory.
- **Faster writes** — inserts/updates to excluded rows do not touch the index.
- **Targeted performance** — speeds up the queries you care about without the overhead of indexing everything.

---

### PostgreSQL: WHERE Clause Syntax

PostgreSQL has first-class support for partial indexes using a `WHERE` clause on `CREATE INDEX`:

```sql
-- Only index active users (assuming 5% of users are active)
CREATE INDEX idx_active_users
  ON users (created_at)
  WHERE status = 'active';
```

This index is 20x smaller than a full index on `created_at` if only 5% of rows match. The planner will use it for queries that include `WHERE status = 'active'`.

**The query's WHERE clause must imply the index predicate** for the planner to consider the partial index:

```sql
-- ✓ Uses the partial index (query predicate matches index predicate)
SELECT * FROM users WHERE status = 'active' AND created_at > '2025-01-01';

-- ✗ Cannot use the partial index (different status value)
SELECT * FROM users WHERE status = 'suspended' AND created_at > '2025-01-01';

-- ✗ Cannot use the partial index (no status filter — might include non-active rows)
SELECT * FROM users WHERE created_at > '2025-01-01';
```

**Common use cases:**

```sql
-- Index only unprocessed jobs (the rows you actually query)
CREATE INDEX idx_pending_jobs ON jobs (priority, created_at) WHERE processed_at IS NULL;

-- Unique constraint only where it matters
CREATE UNIQUE INDEX idx_unique_active_email
  ON users (email)
  WHERE deleted_at IS NULL;

-- Soft deletes: index only non-deleted rows
CREATE INDEX idx_orders_active ON orders (customer_id, order_date) WHERE is_deleted = false;
```

---

### MongoDB: partialFilterExpression

MongoDB supports partial indexes through `partialFilterExpression`:

```javascript
// Only index active users
db.users.createIndex(
  { created_at: 1 },
  { partialFilterExpression: { status: "active" } }
);
```

**Limitations compared to PostgreSQL:**

- The filter expression supports a limited set of operators: `$exists`, `$gt`, `$gte`, `$lt`, `$lte`, `$type`, `$eq`, and `$and`.
- No support for `$ne`, `$in`, `$or`, or regex in the filter expression.
- The query must include a condition that matches the `partialFilterExpression` for the index to be used — same rule as PostgreSQL.

```javascript
// Unique partial index: enforce unique email only for non-deleted users
db.users.createIndex(
  { email: 1 },
  {
    unique: true,
    partialFilterExpression: { deleted_at: { $exists: false } }
  }
);
```

---

### DynamoDB: No Equivalent

DynamoDB **does not support partial indexes**. A GSI indexes every item in the table that has the GSI's key attributes (items missing the key attributes are excluded, which is called a **sparse index** — but this is item-presence-based, not predicate-based).

**Workaround — sparse index pattern:**

```
# Only items with the "active_status" attribute appear in the GSI.
# To "remove" an item from the index, delete the attribute from the base item.

GSI key: active_status (HASH), created_at (RANGE)

# Active user: { pk: "USER#1", active_status: "ACTIVE", created_at: "..." }  → IN GSI
# Inactive user: { pk: "USER#2", created_at: "..." }                         → NOT IN GSI (no active_status attribute)
```

This is a manual pattern, not a declarative feature. It requires application-level discipline to add/remove the attribute.

---

## Combining Partial + Covering Indexes in PostgreSQL

PostgreSQL allows you to combine `WHERE` (partial) with `INCLUDE` (covering) to create highly optimized, surgical indexes:

```sql
-- Scenario: "Show me the email and name of all active users, ordered by signup date"
-- Only 5% of users are active.

CREATE INDEX idx_active_users_covering
  ON users (created_at)
  INCLUDE (email, display_name)
  WHERE status = 'active';
```

**What this achieves:**

1. **Partial:** Only 5% of rows are indexed → small index, fits in memory.
2. **Covering:** `email` and `display_name` are included → Index-Only Scan, no heap access.
3. **Sorted:** `created_at` is the key column → satisfies `ORDER BY` without a sort step.

```sql
-- This query hits the combined index perfectly:
SELECT email, display_name
  FROM users
  WHERE status = 'active'
  ORDER BY created_at DESC
  LIMIT 50;

-- Plan: Index Only Scan Backward using idx_active_users_covering
--       Heap Fetches: 0 (assuming VACUUM is caught up)
```

**Real-world example — background job queue:**

```sql
-- Only pending jobs matter for the worker query.
-- Include payload so the worker never touches the heap.
CREATE INDEX idx_pending_jobs_covering
  ON jobs (priority DESC, created_at ASC)
  INCLUDE (payload, retry_count)
  WHERE status = 'pending';

-- Worker query: fully covered, partial, sorted by priority
SELECT payload, retry_count
  FROM jobs
  WHERE status = 'pending'
  ORDER BY priority DESC, created_at ASC
  LIMIT 1
  FOR UPDATE SKIP LOCKED;
```

---

## Mental Model: Full vs Partial vs Covering vs Partial+Covering

```
                    ┌──────────────────────────────────────────┐
                    │            Which rows indexed?           │
                    ├───────────────────┬──────────────────────┤
                    │    ALL rows       │    SUBSET of rows    │
┌───────────────────┼───────────────────┼──────────────────────┤
│ Must visit table  │   Full Index      │   Partial Index      │
│ for data          │                   │                      │
│                   │   Standard B-tree │   WHERE status='...' │
│                   │   on (col)        │   on (col)           │
├───────────────────┼───────────────────┼──────────────────────┤
│ Query answered    │  Covering Index   │ Partial + Covering   │
│ from index alone  │                   │                      │
│                   │  INCLUDE (col2)   │  WHERE status='...'  │
│                   │  on (col)         │  INCLUDE (col2)      │
│                   │                   │  on (col)            │
└───────────────────┴───────────────────┴──────────────────────┘

  Size:    Full > Covering ≈ Full > Partial > Partial+Covering
  Speed:   Partial+Covering > Covering > Partial > Full
           (for queries that match the partial predicate)
```

**Decision guide:**

| Question | If Yes → |
|----------|----------|
| Do I only query a small subset of rows? | Add a partial predicate (`WHERE`) |
| Does my query need columns beyond the key? | Add `INCLUDE` columns (covering) |
| Both? | Combine — this is the sweet spot |
| Neither? | A regular index is fine |

---

## Support Matrix

| Feature | PostgreSQL | DynamoDB | MongoDB |
|---------|-----------|----------|---------|
| **Covering index** | Yes — `INCLUDE` clause (PG 11+) or all columns as key columns | Yes — GSI with `INCLUDE` or `ALL` projection | Yes — compound index with all queried fields |
| **INCLUDE (non-key payload)** | Yes — `INCLUDE (col)` | Yes — `INCLUDE` projection type | No — all covered fields must be key columns |
| **Partial index** | Yes — `WHERE` clause | No — sparse index is a manual workaround | Yes — `partialFilterExpression` |
| **Partial + covering** | Yes — `WHERE` + `INCLUDE` | No | Partial — compound index with `partialFilterExpression` (no INCLUDE separation) |
| **Heap/fetch bypass** | Index-Only Scan (depends on visibility map) | Always — GSI is a separate copy; no "fallback fetch" | Covered query — IXSCAN with no FETCH stage |
| **Visibility / staleness caveat** | Yes — visibility map + VACUUM dependency | Yes — GSIs are eventually consistent | No equivalent caveat |
| **Unique partial index** | Yes — `CREATE UNIQUE INDEX ... WHERE` | No | Yes — `unique: true` + `partialFilterExpression` |
| **Max secondary indexes** | No hard limit (practical: dozens) | 20 GSIs, 5 LSIs | No hard limit (practical: dozens) |

---

## Key Takeaways

1. **Covering indexes eliminate the most expensive I/O.** The heap/collection fetch is where most query time goes. If your index contains everything the query needs, you skip it entirely.

2. **PostgreSQL's `INCLUDE` is underused.** It lets you add payload columns to an index without bloating the B-tree's internal nodes. Always prefer `INCLUDE` over adding columns as key columns when they are only needed in the `SELECT` list.

3. **PostgreSQL covering indexes depend on VACUUM.** The visibility map must be up to date for Index-Only Scans to avoid heap fetches. On high-write tables, tune autovacuum aggressively.

4. **Partial indexes are PostgreSQL's secret weapon.** If your hot queries only touch a fraction of the table (active users, pending jobs, recent orders), a partial index is smaller, faster, and cheaper to maintain.

5. **DynamoDB has no partial index.** The sparse index pattern is a workaround that requires application-level attribute management. Factor this into your DynamoDB data modeling.

6. **MongoDB requires `_id: 0` in projections for covered queries.** This is the most common reason a MongoDB query hits the collection when it should not.

7. **The partial + covering combination is the endgame in PostgreSQL.** A small index that answers the query without touching the heap — this is the most efficient index you can build.

---

## Interview Talking Points

- **"How do you optimize a slow query?"** — First, check if the query can be answered from an index alone. If the index has the filter columns but the query also needs non-key columns, add them with `INCLUDE` to enable an Index-Only Scan. If only a small subset of rows is queried, make it a partial index. The combination of partial + covering is often the single biggest performance win.

- **"What is the trade-off of a covering index?"** — You trade write performance and storage for read performance. Every `INCLUDE` column is stored in the index leaf pages, increasing index size and write amplification. The trade-off is worth it when reads vastly outnumber writes on the covered query path.

- **"How does DynamoDB handle covering indexes differently?"** — A DynamoDB GSI is a physically separate, eventually consistent copy of the data. You choose which attributes to project at GSI creation time, and you cannot change them without recreating the GSI. Unlike PostgreSQL, there is no transparent fallback — if an attribute is not projected, it is simply not returned.

- **"Why would you use a partial index?"** — When your queries consistently filter to a small subset (e.g., `WHERE status = 'pending'` on a queue table where 99% of rows are completed). The index is smaller, fits in memory, and is cheaper to maintain. PostgreSQL and MongoDB support this natively; DynamoDB requires the sparse index workaround.

- **"In MongoDB, how do you verify a query is covered?"** — Run `.explain("executionStats")` and look for two things: the winning plan has `IXSCAN` with no `FETCH` stage, and `totalDocsExamined` is zero. The most common mistake is forgetting to exclude `_id` from the projection.
