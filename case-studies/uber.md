# Case Study: Uber Ride-Sharing System Design

## Problem Overview

Uber looks simple on the surface — a rider requests a ride, a driver picks them up. But the system behind it is one of the hardest real-time matching problems in distributed systems:

- **Real-time geospatial tracking** at massive scale — millions of drivers broadcasting locations continuously.
- **Strong consistency under concurrency** — two matching workers must never assign the same driver to two riders.
- **Fault tolerance with time pressure** — a rider waiting 60 seconds for a match cannot tolerate a lost request due to a crashed worker.
- **Global scale with local latency** — the system operates across dozens of countries, but each match must feel instant.

The difficulty is not in any single feature — it is in the intersection of low latency, strong consistency, high throughput, and fault tolerance, all operating on rapidly changing geospatial data.

---

## Functional Requirements

### Above the Line (Core Scope)

1. **Fare Estimation** — Rider enters pickup and dropoff locations, receives an estimated fare before committing.
2. **Ride Requesting** — Rider confirms the fare and requests a ride.
3. **Driver Matching** — System finds the nearest available driver and assigns them to the ride.
4. **Driver Accept/Decline** — Matched driver can accept or decline; if declined, the system re-matches.
5. **Navigation** — Driver receives turn-by-turn directions to the rider and then to the destination.

### Below the Line (Out of Scope for Interview)

- Ratings and reviews
- Scheduled rides
- Ride categories (UberX, UberXL, UberBlack)
- Payment processing
- Driver onboarding and background checks

---

## Non-Functional Requirements

### Above the Line

| NFR | Target | Why It Matters |
|-----|--------|----------------|
| **Low Latency Matching** | < 1 minute from request to driver assignment, or fail gracefully | Riders abandon after ~60s; a slow match is a failed match |
| **Strong Consistency** | No double-assignment of a driver to two rides | Double-booking wastes rider time and erodes trust |
| **High Throughput** | 100k+ concurrent ride requests during peak | Friday night, New Year's Eve, concert endings — traffic is extremely spiky |

### Below the Line

- GDPR compliance for location data
- CI/CD pipeline for rapid deployment
- Monitoring and alerting (P99 latency, match success rate)
- Multi-region disaster recovery

---

## Core Entities

```
┌──────────┐     ┌──────────────┐     ┌──────────┐
│  Rider   │────▶│   Fare       │────▶│   Ride   │
│          │     │              │     │          │
│ id       │     │ id           │     │ id       │
│ name     │     │ riderId      │     │ fareId   │
│ email    │     │ pickup       │     │ driverId │
│ location │     │ dropoff      │     │ status   │
└──────────┘     │ amount       │     │ pickup   │
                 │ currency     │     │ dropoff  │
┌──────────┐     │ estimateAt   │     │ startedAt│
│  Driver  │     │ surgeMulti   │     │ endedAt  │
│          │     │ expiresAt    │     └──────────┘
│ id       │     └──────────────┘
│ name     │     ┌──────────┐
│ status   │     │ Location │
│ location │     │          │
│ vehicleInfo│   │ driverId │
└──────────┘     │ lat, lng │
                 │ timestamp│
                 └──────────┘
```

### Why Fare and Ride Are Separate Entities

A fare is an **estimate** — it exists the moment a rider checks a price, even if they never request a ride. A ride is a **commitment** — it only exists after the rider confirms and the system begins matching.

Separating them means:

- **Analytics**: You can track fare-to-ride conversion rates (how many estimates convert to actual rides).
- **Pricing isolation**: Fare calculation logic (surge, distance, currency) stays decoupled from ride lifecycle logic (matching, navigation, completion).
- **Auditability**: The original estimate is preserved even if the final charge differs (route change, traffic, etc.).

---

## API Design

### 1. Estimate a Fare

```
POST /fare
```

**Request Body:**
```json
{
  "pickupLocation": { "lat": 40.7128, "lng": -74.0060 },
  "dropoffLocation": { "lat": 40.7580, "lng": -73.9855 }
}
```

**Response:** `201 Created`
```json
{
  "fareId": "fare_abc123",
  "estimatedAmount": 24.50,
  "currency": "USD",
  "estimatedDuration": "18 min",
  "estimatedDistance": "5.2 mi"
}
```

Rider identity comes from the JWT in the `Authorization` header — not from the request body.

### 2. Request a Ride

```
POST /rides
```

**Request Body:**
```json
{
  "fareId": "fare_abc123"
}
```

The body contains **only the fareId**. Pickup, dropoff, and pricing are already stored with the fare. This prevents the client from tampering with locations or fare amounts.

**Response:** `201 Created`
```json
{
  "rideId": "ride_xyz789",
  "status": "MATCHING"
}
```

### 3. Update Driver Location

```
POST /drivers/location
```

**Request Body:**
```json
{
  "lat": 40.7135,
  "lng": -74.0046
}
```

Driver identity comes from the JWT — **never from the request body**. This prevents a malicious client from spoofing another driver's location.

### 4. Update Ride Status (Driver Accept/Decline/Complete)

```
PATCH /rides/:rideId
```

**Request Body:**
```json
{
  "action": "ACCEPT"
}
```

Valid actions: `ACCEPT`, `DECLINE`, `START`, `COMPLETE`.

### Security Notes

> **Never trust client-supplied identity or fare values.**
> - User identity always comes from the JWT (set by the API Gateway after authentication).
> - Fare amounts are server-calculated and stored server-side. The client only sends a `fareId` reference.
> - Location updates are tied to the authenticated driver — no `driverId` in the body.

---

## Service Architecture

```
                              ┌─────────────────────┐
                              │  Third-Party Map API │
                              │  (Google/Mapbox)     │
                              └──────────▲───────────┘
                                         │
┌─────────┐    ┌──────────────┐    ┌─────┴──────────┐    ┌──────────────┐
│  Rider   │───▶│              │───▶│  Ride Service  │───▶│  PostgreSQL  │
│  App     │    │              │    │                │    │  (fares,     │
└─────────┘    │  API Gateway │    └────────┬───────┘    │   rides)     │
               │              │             │            └──────────────┘
┌─────────┐    │  • Auth      │    ┌────────▼───────┐    ┌──────────────┐
│  Driver  │───▶│  • Rate Limit│───▶│Location Service│───▶│    Redis     │
│  App     │    │  • Routing   │    │                │    │  (GEOADD /   │
└─────────┘    └──────────────┘    └────────────────┘    │   GEOSEARCH) │
                                            │            └──────────────┘
                                   ┌────────▼───────┐
                                   │     Kafka      │
                                   │ (ride-requests │
                                   │     topic)     │
                                   └────────┬───────┘
                                            │
                                   ┌────────▼───────┐    ┌──────────────┐
                                   │Matching Worker │───▶│    Redis      │
                                   │  (consumer     │    │  (GEOSEARCH  │
                                   │   group)       │    │  + SET NX    │
                                   └────────┬───────┘    │  dist. lock) │
                                            │            └──────────────┘
                                   ┌────────▼───────┐
                                   │  Notification  │───▶  APNs / FCM
                                   │  Service       │    (push notifications)
                                   └────────────────┘

                                   ┌────────────────┐    ┌──────────────┐
                                   │ Exchange Rate  │───▶│    Redis      │
                                   │ Fetcher (cron) │    │ (currency    │
                                   └────────────────┘    │   cache)     │
                                                         └──────────────┘
                                   ┌────────────────┐    ┌──────────────┐
                                   │Surge Calculator│◀──▶│    Redis      │
                                   │ (background    │    │ (driver pos, │
                                   │  job, 30-60s)  │    │  req counts, │
                                   └────────────────┘    │  surge mult) │
                                                         └──────────────┘
```

### Service Responsibilities

| Service | Responsibility | Storage |
|---------|---------------|---------|
| **API Gateway** | Authentication, rate limiting, request routing | — |
| **Ride Service** | Fare estimation, ride lifecycle (create, update status) | PostgreSQL |
| **Location Service** | Ingest driver GPS pings, expose geospatial queries | Redis (geospatial sorted set) |
| **Kafka** | Decouple ride requests from matching; buffer during spikes | — |
| **Matching Worker** | Find nearest available driver, acquire lock, assign | Redis (GEOSEARCH + SET NX) |
| **Notification Service** | Push ride offers to drivers, status updates to riders | APNs / FCM |
| **Exchange Rate Fetcher** | Periodically refresh currency exchange rates | Redis (cache) |
| **Surge Calculator** | Compute demand/supply ratio per zone, update surge multipliers every 30-60s | Redis (read driver positions + request counters, write surge multipliers) |

---

## Three Hard Problems

### Hard Problem 1: Real-Time Location at Scale

#### The Scale

```
10 million active drivers × 1 ping every 5 seconds = 2,000,000 writes/sec
```

This is not a read-heavy problem. It is an extreme **write-heavy, geospatial** problem.

#### Why a Standard Database Fails

- **Write volume**: 2M writes/sec saturates any disk-backed database, even with SSDs. PostgreSQL's WAL and MVCC overhead make this infeasible.
- **B-tree inefficiency for geospatial**: Standard B-tree indexes work on one dimension. Finding "drivers within 3 km" requires searching two dimensions (lat + lng). A composite `(lat, lng)` index can range-scan on `lat` but must then linearly filter `lng` — poor performance.
- **Constant mutation**: Driver locations change every 5 seconds. B-tree rebalancing on every update creates write amplification.

#### Solution: Redis GEOADD + GEOSEARCH

Redis stores geospatial data using a **sorted set with geohash encoding**. A geohash converts 2D coordinates into a 1D string by interleaving latitude and longitude bits, creating a space-filling curve where nearby points share common prefixes.

```redis
-- Store driver location (O(log N) — sorted set insertion)
GEOADD drivers:nyc -74.0060 40.7128 driver_123

-- Find drivers within 3 km of rider (O(N+log(M)) — M is sorted set size)
GEOSEARCH drivers:nyc FROMLONLAT -74.0050 40.7130 BYRADIUS 3 km ASC COUNT 10
```

**Why this works:**
- Redis is in-memory — no disk I/O for writes.
- Geohash encoding means "find nearby" is a **prefix range scan** on a sorted set, not a 2D spatial query.
- Sorted sets handle constant updates efficiently (O(log N) per update).

#### Optimizations

**Adaptive update intervals**: Not all drivers need to ping every 5 seconds.

```
Stationary driver (parked)     → ping every 30s
Moving driver (on a trip)      → ping every 5s
Driver approaching pickup      → ping every 2s
```

This reduces write volume by 50-70% without meaningfully degrading matching accuracy.

**Stale driver cleanup**: If a driver has not pinged in 30 seconds, assume they are offline (app crashed, phone died, lost signal). A background job removes stale entries:

```redis
-- Application-level: check timestamp before considering a driver
-- If driver_123's last ping was > 30s ago, skip them in matching
```

#### Connection to NFRs

- **Low latency matching**: GEOSEARCH returns nearest drivers in microseconds.
- **High throughput**: Redis handles millions of writes/sec on a single node.

---

### Hard Problem 2: Strong Consistency on Driver Assignment

#### The Problem

The matching worker is horizontally scaled (multiple instances consuming from Kafka). Two instances could simultaneously:

1. Find that Driver A is the nearest available driver.
2. Both assign Driver A to different rides.

Result: **double-booking** — Driver A gets two ride notifications, one rider gets ghosted.

#### Bad Solution: Application-Level Locking

```python
# ❌ Each worker instance has its own lock — no coordination
if driver not in self.locked_drivers:   # in-memory check
    self.locked_drivers.add(driver)
    assign_driver(driver, ride)
```

This fails because each worker process has its own memory. Worker A's lock is invisible to Worker B.

#### Good Solution: Database Status Update

```sql
-- Atomically update driver status only if they are still available
UPDATE drivers
SET status = 'ASSIGNED', ride_id = 'ride_xyz'
WHERE id = 'driver_123' AND status = 'AVAILABLE';

-- Check rows affected: if 0, another worker already claimed this driver
```

This is correct — the database enforces atomicity. But there are downsides:

- **Write pressure on PostgreSQL**: Every match attempt hits the database.
- **In-memory timeout risk**: If the matching worker crashes after acquiring the assignment but before sending the notification, the driver is stuck in `ASSIGNED` forever without a cleanup mechanism.

#### Great Solution: Redis SET NX with TTL (Distributed Lock)

```redis
-- Try to acquire a lock on the driver (atomic, with 10s expiry)
SET driver_lock:driver_123 ride_xyz NX EX 10
```

- `NX` — only set if the key does **not** already exist (atomic compare-and-set).
- `EX 10` — auto-expires after 10 seconds (built-in timeout, prevents deadlocks).

```python
# Matching Worker pseudocode
def match_ride(ride):
    nearby_drivers = redis.geosearch("drivers:nyc", ride.pickup, radius=3km)

    for driver in nearby_drivers:
        # Attempt to acquire distributed lock
        acquired = redis.set(f"driver_lock:{driver.id}", ride.id, nx=True, ex=10)

        if acquired:
            # We have exclusive access to this driver
            notify_driver(driver, ride)
            update_ride_status(ride.id, "DRIVER_NOTIFIED")
            return

    # No driver available within radius
    expand_search_or_fail(ride)
```

**Why this is better:**
- **Atomic**: `SET NX` is a single Redis command — no race condition.
- **Self-healing**: The 10s TTL means if a worker crashes, the lock auto-releases. No manual cleanup needed.
- **Fast**: Redis responds in sub-millisecond; no database round-trip for locking.

#### Connection to Optimistic vs Pessimistic Locking

This Redis distributed lock is conceptually **pessimistic** — you acquire exclusive access before proceeding. But unlike traditional pessimistic locks (`SELECT FOR UPDATE`), it has a **built-in timeout** (TTL), combining the safety of pessimistic locking with the liveness guarantee of optimistic approaches.

See also: [database-selection.md](../databases/database-selection.md) — Concurrency Support section.

#### Connection to NFRs

- **Strong consistency**: SET NX guarantees exactly one worker can claim a driver.
- **Low latency**: Sub-millisecond lock acquisition.

---

### Hard Problem 3: Fault Tolerance — No Dropped Requests

#### The Problem

During peak load (100k+ concurrent requests), a matching worker crash means all in-flight ride requests assigned to that worker are **lost**. The rider sees a spinner forever.

Without durability, the system trades reliability for speed — and loses both when things go wrong.

#### Solution Part 1: Kafka as a Durable Queue

Instead of the Ride Service directly calling the Matching Worker, it publishes to a Kafka topic:

```
Ride Service  →  Kafka (ride-requests topic)  →  Matching Worker (consumer group)
```

**Key behavior**: The matching worker commits its Kafka offset **only after a successful match**.

```python
def consume_ride_requests():
    for message in kafka_consumer:
        ride = parse(message)

        success = match_ride(ride)

        if success:
            kafka_consumer.commit()  # Offset advances
        else:
            # Don't commit — Kafka will re-deliver this message
            # after consumer timeout (another worker picks it up)
            pass
```

If a worker crashes mid-match:
1. The offset was never committed.
2. Kafka reassigns the partition to another worker in the consumer group.
3. The new worker re-processes the ride request.
4. **No ride is lost.**

#### Solution Part 2: Temporal Durable Workflows for Driver Timeout

What happens when a driver is matched but does not respond within 10 seconds? You need a **durable timer** that survives worker crashes.

```
┌─────────────────────────────────────────────────┐
│           Temporal Durable Workflow              │
│                                                  │
│  1. Send offer to Driver A                       │
│  2. Start 10s durable timer                      │
│  3. If accepted → ride confirmed                 │
│  4. If timeout  → release lock, try Driver B     │
│  5. If timeout  → release lock, try Driver C     │
│  6. If all fail → notify rider "no drivers"      │
│                                                  │
└─────────────────────────────────────────────────┘
```

**Why Temporal, not setTimeout?**

- `setTimeout` or cron lives in a single process. If that process dies, the timer dies.
- Temporal persists workflow state to a database. If the worker crashes, another worker picks up the workflow exactly where it left off — timer included.
- The driver timeout chain (A → B → C) is expressed as linear code, not a state machine or callback hell.

#### Solution Part 3: Geo-Sharding for Global Scale

At global scale, a single Kafka cluster and Redis instance cannot handle all regions. Partition by geography:

```
┌──────────────┐  ┌──────────────┐  ┌──────────────┐
│  US-East     │  │  EU-West     │  │  APAC        │
│  Kafka       │  │  Kafka       │  │  Kafka       │
│  Redis       │  │  Redis       │  │  Redis       │
│  Workers     │  │  Workers     │  │  Workers     │
└──────────────┘  └──────────────┘  └──────────────┘
```

- Rides in New York are matched against drivers in the US-East shard.
- No cross-region coordination needed — a ride in NYC will never match a driver in London.
- Each shard scales independently based on regional demand.

#### Connection to NFRs

- **Fault tolerance**: Kafka prevents lost requests; Temporal prevents stuck workflows.
- **High throughput**: Consumer groups scale horizontally; geo-sharding distributes load.

---

## Surge Pricing

### What It Is

Dynamic fare multiplier applied when demand exceeds supply in a geographic zone.

```
surge_fare = base_fare × surge_multiplier
surge_multiplier = f(pending_requests, available_drivers, zone)
```

### Four Hard Problems

1. **Detect demand/supply imbalance in real time** — requires counting requests and drivers per zone continuously.
2. **Calculate and update multiplier consistently** — a background job must refresh multipliers across all zones every 30-60 seconds.
3. **Honor the price the rider was shown** — the multiplier at fare estimation time must be the multiplier at confirmation time.
4. **Prevent race condition between estimation and confirmation** — surge can change between the two API calls.

### Zone Division — Two Approaches

- **Geohash precision 5** (~5 km × 5 km cells) — natural fit with Redis key naming (e.g., `zone:surge:dr5ru`).
- **H3 hexagonal grid** (Uber's actual approach) — more uniform cells, better boundary coverage. Worth mentioning at staff level.

### Measuring Supply and Demand Per Zone

- **Supply**: `GEOSEARCH` within zone radius → count available drivers.
- **Demand**: Redis counter per zone (`zone:requests:{geohash}`), incremented on ride request, decremented on match.

```javascript
const ratio = pendingRequests / availableDrivers;
```

### Multiplier Calculation (Background Job, Runs Every 30-60s)

```
ratio < 1.0  →  1.0x  (no surge)
ratio < 1.5  →  1.2x
ratio < 2.0  →  1.5x
ratio < 3.0  →  1.8x
ratio >= 3.0 →  2.5x  (cap to prevent gouging)
```

Store in Redis with TTL 60s:

```redis
SET zone:surge:{geohash} {multiplier} EX 60
```

This is the **materialized view pattern** applied to surge: a pre-computed ratio cached in Redis, refreshed periodically.

### Applying Multiplier at Fare Estimation (Ride Service)

- Read `zone:surge:{geohash}` from Redis at `POST /fare` time.
- Snapshot the multiplier onto the Fare entity in PostgreSQL.
- Include `surgeExpiresAt` field on the Fare response.

### The Hardest Problem — Price Consistency

**Race condition:**

```
T=0:   Rider sees $10 (1.0x multiplier)
T=15:  Surge kicks in → 2.0x
T=20:  Rider confirms → pays $10 or $20?
```

**Solution:** Snapshot the multiplier at estimation time, honor it at confirmation time. The Fare entity stores a `surge_multiplier` column. `POST /rides` reads the fare from the database — it never recalculates.

Add a fare expiry window (2-5 minutes):

```
if fare.expiresAt < now → 409 Conflict, rider must re-request
```

**Fare entity additions:**

```
surge_multiplier   DECIMAL     -- snapshotted at estimation
expires_at         TIMESTAMP   -- confirmation deadline
```

### Surge Heatmap — The One CDN Use Case in Uber

- **Option 1**: API returns `[{geohash, multiplier}]`, client renders overlay.
- **Option 2**: Pre-render heatmap as image tiles → S3 → CloudFront CDN.
  - Tiles are identical for every rider in the same viewport.
  - Refreshed every 60 seconds.
  - This is the legitimate CDN use case for Uber.

### How Surge Connects to Existing Architecture

| Component | Surge Pricing Role |
|-----------|--------------------|
| **Redis GEOADD** | Source of driver supply data |
| **Redis cache** | Stores `zone:surge:{geohash}` multipliers |
| **Location Service** | Feeds supply data into surge calculation |
| **Ride Service** | Reads + snapshots multiplier at `POST /fare` |
| **Fare (PostgreSQL)** | Stores multiplier snapshot + expiry |
| **Matching Worker** | Unchanged |
| **CDN** | Serves pre-rendered surge heatmap tiles |

Surge Calculator is a **background job**, not a new microservice.

---

## Does Uber Need a CDN?

**Short answer: Not for the core flows.**

Every core interaction — fare estimation, ride requesting, location updates, matching — is a **dynamic API call** with personalized, real-time data. You cannot cache "find me the nearest driver" at an edge node.

**Where a CDN does apply:**

| Use Case | Cacheable? | Refresh Frequency | CDN Value |
|----------|-----------|-------------------|-----------|
| **Map tiles** | Yes | Rarely changes | High — self-hosted map rendering tiles are static and highly cacheable |
| **Static app assets** | Yes | Per deploy | Standard — JS bundles, images, fonts for web dashboard |
| **Driver/rider profile images** | Yes | Rarely changes | Standard — static media |
| **Surge heatmap tiles** | Yes | Every 60s | High — pre-rendered image tiles identical for all riders in same viewport, served via S3 → CloudFront |

**The interviewer takeaway:** Knowing when **not** to use a technology is as valuable as knowing when to use it. CDNs solve latency for static, cacheable content. Uber's core problem is dynamic, real-time data — a CDN does not help there. The one interesting exception is surge heatmap tiles: the surge state is semi-static (refreshed every 60s) and identical for all viewers in a region, making it a legitimate CDN use case.

---

## Deep Dive to NFR Mapping

| Deep Dive | Primary NFR | How It Satisfies the NFR |
|-----------|-------------|--------------------------|
| Redis GEOADD/GEOSEARCH | Low Latency Matching | Sub-ms geospatial queries vs seconds with a traditional DB |
| Adaptive ping intervals | High Throughput | Reduces write volume 50-70%, preventing Redis saturation |
| Redis SET NX distributed lock | Strong Consistency | Atomic lock prevents double-assignment across workers |
| Kafka with offset commit | Fault Tolerance | Crashed workers do not lose ride requests |
| Temporal durable workflows | Low Latency Matching + Fault Tolerance | Driver timeout chain continues even if worker crashes |
| Geo-sharding | High Throughput | Horizontal partitioning eliminates global bottleneck |

---

## Interview Delivery Strategy

### Time Budget (45-Minute Interview)

```
┌─────────────────────────────────────────────────────┐
│  Requirements & Scoping        │  5 min             │
│  Core Entities & API Design    │  5 min             │
│  High-Level Architecture       │ 15 min             │
│  Deep Dives (3 hard problems)  │ 20 min             │
└─────────────────────────────────────────────────────┘
```

### How to Flag Problems Proactively

Do not wait for the interviewer to ask "what about consistency?" — raise it yourself:

> "Before I go deeper into the architecture, I want to flag three hard problems I see: real-time geospatial tracking at 2M writes/sec, the double-assignment race condition in matching, and fault tolerance under peak load. I'd like to deep-dive into each of these."

This demonstrates **architectural awareness** — you see the hard parts before being told to look for them.

### Level Expectations

| Level | Expectation |
|-------|-------------|
| **Mid-Level** | Correct functional design, reasonable API, identifies that matching is hard |
| **Senior** | All of the above + solves at least one hard problem well (e.g., Redis for geospatial), mentions trade-offs |
| **Staff+** | All of the above + solves all three hard problems, connects each solution to a specific NFR, discusses operational concerns (monitoring, failure modes, geo-sharding), and explains what they would **not** build (e.g., CDN for core flows) |

---

## Key Takeaways

1. **Uber is a real-time matching problem, not a CRUD app.** The difficulty is in the intersection of geospatial data, consistency, and fault tolerance — not in any single feature.

2. **Redis is the backbone.** It handles three distinct roles: geospatial indexing (GEOADD/GEOSEARCH), distributed locking (SET NX), and caching (exchange rates). Understand why Redis fits each role.

3. **Consistency and availability are in tension.** The SET NX lock trades availability (a locked driver cannot be matched elsewhere) for consistency (no double-booking). This is a deliberate choice.

4. **Kafka decouples and protects.** Without Kafka, a crashed matching worker loses ride requests. With Kafka, requests are durable and automatically reprocessed.

5. **Temporal handles the messy middle.** The driver timeout chain (offer → wait → fallback) is inherently stateful and must survive crashes. Temporal makes durable workflows feel like regular code.

6. **Know when to say no.** CDNs, caching layers, and other standard tools do not help with Uber's core problem. Saying "we don't need a CDN here" is a stronger signal than adding one reflexively.

---

## Interview Talking Points

- **"Why Redis over PostgreSQL for locations?"** — 2M writes/sec of constantly mutating geospatial data. PostgreSQL's WAL, MVCC overhead, and B-tree rebalancing cannot keep up. Redis is in-memory with O(log N) geohash updates — purpose-built for this workload.

- **"Why not just use a database lock for driver assignment?"** — A database lock works for correctness, but adds write pressure to PostgreSQL for every match attempt. Redis SET NX is sub-millisecond, atomic, and self-healing via TTL. It keeps the hot path off the database.

- **"What happens if a matching worker crashes?"** — Kafka offset was never committed, so the partition is reassigned to another consumer. The ride request is reprocessed automatically. If the crash happens after the lock was acquired but before the notification, the TTL expires in 10 seconds and the driver becomes available again.

- **"How do you handle a driver who never responds?"** — A Temporal durable workflow sends the offer, starts a 10s timer, and if no response, releases the lock and tries the next nearest driver. This chain survives worker crashes because Temporal persists workflow state.

- **"How does this scale globally?"** — Geo-sharding. Each region (US-East, EU-West, APAC) has its own Kafka cluster, Redis instance, and matching workers. A ride in NYC never needs to coordinate with infrastructure in London. Each shard scales independently.

- **"How does surge pricing work?"** — A background job runs every 30-60s, counts pending requests vs available drivers per geohash zone, and writes a multiplier to Redis with a 60s TTL. At fare estimation time, the Ride Service reads the multiplier and snapshots it onto the Fare entity. At confirmation time, the system honors the snapshotted price — it never recalculates. A 2-5 minute expiry window on the fare prevents stale prices from being used. This is the materialized view pattern applied to pricing.

- **"What would you monitor in production?"** — Match latency P50/P95/P99, match success rate, Kafka consumer lag (are we falling behind?), Redis memory usage, stale driver count, Temporal workflow failure rate, and surge multiplier distribution per zone (detect runaway surge or stuck calculations).

---

## Cross-References

- See also: [indexes.md](../databases/indexes.md) — Redis geospatial internals and sorted set indexing
- See also: [consistency.md](../distributed-systems/consistency.md) — Distributed locks, optimistic vs pessimistic locking
- See also: [caching/](../caching/) — Redis caching patterns and TTL strategies
- See also: [database-selection.md](../databases/database-selection.md) — Why PostgreSQL for rides/fares, concurrency support
