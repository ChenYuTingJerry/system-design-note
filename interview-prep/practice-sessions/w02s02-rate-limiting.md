# System Design Practice — Week 2, Session 2



## Topic: Rate Limiting System



**Date:** 2026-04-30

**Score:** 7.5 / 10

**Phase:** Introductory (Week 2)



---



## What Is a Rate Limiter?



A rate limiter is a server-side mechanism that controls the number of requests a client can make to an API within a specific time window. It protects infrastructure from abuse, prevents resource exhaustion, and ensures fair usage across all users.



**Why companies build rate limiters:**



- Prevent abuse and DDoS attacks from overwhelming services

- Protect downstream services (databases, queues) from traffic spikes

- Ensure fair resource allocation across users

- Maintain SLA compliance for latency and availability



**What a successful rate limiter looks like:**



| Metric | Before Rate Limiter | After Rate Limiter |

|--------|--------------------|--------------------|

| Abuse-related outages | Frequent | Rare |

| Legitimate user blocking | N/A | < 0.1% of requests |

| API response time during spikes | Degraded (timeouts) | Stable (< 200ms) |

| Infrastructure cost during attacks | Spikes unpredictably | Controlled |



---



## Scenario



> You're designing a rate limiter for a public API that handles 10,000 requests per second at peak load. The API serves both authenticated users (with API keys) and unauthenticated public traffic. Your system needs to prevent abuse while not blocking legitimate users, especially those behind corporate proxies or carrier-grade NAT.



### Clarified Constraints



- **Latency budget:** Under 5 milliseconds for rate limit check

- **Failure strategy:** Fail-open (let requests through if rate limiter crashes)

- **Consistency:** Eventual consistency acceptable across servers

- **Storage:** Can use Redis

- **Timeline:** 3-month MVP

- **Architecture:** Centralized rate limiter shared by all API services

- **Rate limit scope:** Varies by endpoint and by user ID/IP

- **Rate limit identity:** Authenticated users by user ID/API key; unauthenticated by IP



---



## My Architecture Decisions



### Component 1: Rate Limiter Middleware



- **Rate limiter sits inside the API Gateway** as middleware

- Every request passes through the rate limiter before reaching backend services

- Flow: Client → API Gateway → Rate Limiter (middleware) → Backend Services



### Component 2: Token Bucket Algorithm



- **Chose token bucket** over sliding window (too much memory) and fixed window (burst problem at window boundaries)

- Token bucket smooths out bursts and is memory-efficient

- Tokens refill continuously (e.g., 150 tokens/min = 2.5 tokens/sec)

- On each request: calculate refill based on elapsed time, check if tokens >= 1, decrement if allowed, return 429 if denied

- **Refill on-the-fly** when request arrives (no background workers needed)



### Component 3: Redis Counter Storage



- **Redis stores token counters** keyed by `endpoint:userID/IP`

- Redis is single-threaded, but race conditions still possible across multiple API Gateway instances

- **Lua scripts** used for atomic read-modify-write operations (check tokens, decrement, refill — all in one atomic operation)

- Lua scripts prevent race conditions because Redis executes the entire script atomically — no interleaving from other clients



### Component 4: Configuration Management



- **Configuration service** stores rate limit rules (endpoint → RPS/RPM)

- **Two-tier caching strategy:**

  1. On API Gateway startup: load all config into memory (one-time job)

  2. Sidecar/background process watches config service for changes and pushes updates to in-memory cache

- On each request: rate limiter checks in-memory cache (no external call) — keeps latency under 5ms

- **Manual fallback API endpoint** on API Gateway for pushing config updates if config service is down



### Component 5: Failure Handling



- **Fail-open strategy:** If Redis is down, log the error and let requests pass through

- **Circuit breaker pattern:** If Redis is down for >X seconds, circuit breaker opens, API Gateway switches to local in-memory rate limiting

- **Alerting mechanism** fires when Redis is down so engineers can respond quickly

- **Config service redundancy:** Primary + backup config service with leader election (Raft/Paxos) for automatic failover



### Component 6: Multi-Region Design



- **Two approaches evaluated:**

  1. **Multi-region Redis cluster** — all regions see same data, strong consistency, high cost

  2. **Shard by region** — separate Redis per region (US, Asia, EU), no cross-region replication, lower cost

- **Chose shard-by-region** as default — cheaper, simpler, sufficient for most APIs where abuse is regional

- Only upgrade to multi-region cluster if global rate limits are required and cost is justified

- Trade-off: users traveling between regions get fresh limits per region (acceptable for most use cases)



### Response Handling



- Return **429 Too Many Requests** with:

  - `Retry-After` header (when to retry)

  - `X-RateLimit-Remaining` (tokens left)

  - `X-RateLimit-Reset` (when window resets)



---



## Risks Identified



1. **Eventual consistency slippage** — During multi-region sync windows, requests may slip through and exceed limits. Mitigate with tighter replication windows and 429 rate monitoring.

2. **Redis outage** — Extended Redis downtime means no rate limiting (fail-open). Mitigate with circuit breaker pattern switching to local in-memory fallback + alerting.

3. **Configuration service failure** — Can't update rate limit rules. Mitigate with config service replication (leader election) and manual API endpoint fallback.

4. **Blocking legitimate users** — Enterprise customers behind shared corporate IPs or carrier-grade NAT. Mitigate by rate limiting by user ID/API key for authenticated users, customizing limits per customer, and monitoring 429 error rates.



---



## Scoring Feedback



### What Went Well



- **Strong constraint discipline** — systematic clarifying questions covering latency budget, fail-open strategy, consistency, Redis availability, multi-region. Major improvement from Session 1.

- **Solid algorithm choice** — token bucket over sliding window and fixed window, with clear reasoning (memory efficiency, burst handling)

- **Real-world awareness** — referenced the Medium article problem (corporate IPs, shared NAT) and showed how to avoid it through customer customization and p99 monitoring

- **Bottleneck thinking** — identified three real risks (consistency slippage, Redis outage, config service failure) with mitigations for each

- **Operational mindset** — monitoring p99 usage, setting limits at 150% of p99, alerting on 429 rates



### What Needs Work



1. **Lua script explanation was shaky** — eventually understood it, but took too long. Practice explaining atomicity: "Lua script runs as one indivisible operation on Redis — no other command can interleave"

2. **Multi-region design felt incomplete** — jumped to "multi-region cluster" without upfront cost analysis. Should have proposed multiple options (shard-by-region vs multi-region cluster) and compared cost vs consistency

3. **Circuit breaker was mentioned but not detailed** — missing: timeout threshold, how local fallback works exactly, when to close the circuit again

4. **Grammar/clarity improving** — better than Session 1, but still some stumbling and trailing off



### Key Gap for Staff Level



> Staff level requires **cost-benefit analysis** in tradeoffs. Example: "I proposed multi-region cluster for global consistency, but that's expensive. For most APIs, shard-by-region is cheaper and sufficient. Only upgrade to multi-region cluster if we have truly global users and strong consistency is critical. The cost difference is significant — separate Redis per region avoids cross-region replication traffic entirely."



---



## Key Learnings from This Session



### Cost-Benefit Tradeoff Framework



Three dimensions to always consider:



1. **Operational cost** — infrastructure, licenses, managed services

2. **Engineering cost** — time to build, maintain, debug

3. **Business cost** — impact if it fails, revenue loss, customer churn



### Multi-Region Rate Limiting Options



| Option | Cost | Consistency | Latency | When to Use |

|--------|------|-------------|---------|-------------|

| Multi-region cluster | High | Strong | Low everywhere | Global users, high-value APIs |

| Single Redis + local cache | Medium | Eventual | Low locally | Most traffic is regional |

| Shard by region | Low | Per-region only | Low everywhere | Abuse is regional, cost-sensitive |

| Local in-memory only | Very low | Per-gateway only | Sub-ms | Temporary fallback during outages |



### How to Pick Rate Limit Numbers



- Pull a week of request logs

- Group by user ID/IP

- Look at p50, p95, p99

- Set limit at **150% of p99** to leave headroom for legitimate bursts

- Monitor 429 rates — if > 0.1% of legitimate traffic is blocked, limit is too tight



---



## Recommendations for Session 3



- Pick a **notification system or data pipeline** — another stretch topic

- Focus on **cost-benefit analysis** in every tradeoff (not just technical, but business impact)

- Practice **explaining distributed algorithms** (Raft, consensus, atomicity) more clearly

- Tighten delivery — less trailing off, more structured sentences

- Time yourself: 5 min clarify, 3 min constraints, 10 min architecture, 5 min deep dive, 5 min scaling/tradeoffs



---



## Progress Tracking



| Session | Topic | Score | Key Improvement |

|---------|-------|-------|-----------------|

| Week 2, Session 1 | Internal Developer Platform | 6.5 / 10 | Baseline — identified framework gaps |

| Week 2, Session 2 | Rate Limiting System | 7.5 / 10 | +1.0 — constraint discipline, bottleneck thinking |
