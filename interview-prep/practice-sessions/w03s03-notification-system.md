# System Design Practice — Week 3, Session 3

## Topic: Notification System

**Date:** 2026-05-05
**Score:** 7.8 / 10
**Phase:** Introductory (Week 3)

---

## What Is a Notification System?

A notification system is a centralized service that manages multi-channel message delivery (email, SMS, push notifications) to users. It decouples notification logic from business services, provides user preference management, ensures reliable delivery with retries and tracking, and integrates with third-party providers for actual message delivery.

**Why companies build notification systems:**

- Eliminate ad-hoc notification sending scattered across services
- Provide consistent delivery guarantees across all channels
- Enable user preference management (opt-in/opt-out per channel)
- Track delivery status and debug failed notifications
- Scale message throughput independently from business logic

**What a successful notification system looks like:**

| Metric | Before | After |
|--------|--------|-------|
| Lost notifications | Frequent (no tracking) | < 0.01% (tracked + retried) |
| Delivery latency (95th percentile) | Unpredictable | < 5 minutes |
| User preference compliance | Manual / inconsistent | 100% automated |
| Duplicate notifications | Common | Near-zero (idempotency) |
| Channel failover time | Manual intervention | Automatic (config-driven) |

---

## Scenario

> You're designing a notification system for a mid-size SaaS platform. The product sends notifications to users across three channels: email, SMS, and push notifications. Right now it's a mess — notifications are sent ad-hoc from different services, some get lost, some arrive late, and there's no way for users to control what they receive.

### Clarified Constraints

- **Users:** 10 million users, 500K daily active
- **Peak load:** 50K notifications per second across all channels
- **SLA:** 95% delivered within 5 minutes, 99% within 1 hour
- **Team size:** 4 engineers, no dedicated ops team
- **Tech stack:** Kubernetes, Python microservices, PostgreSQL, Redis
- **Protocol:** REST API (synchronous client submission, asynchronous delivery)
- **Timeout:** 24-72 hours depending on channel before giving up
- **Retry:** Exponential backoff with dead letter queue for persistent failures
- **Third-party providers:** Managed services (SendGrid for email, Twilio for SMS)

---

## My Architecture Decisions

### Component 1: API Gateway

- **API Gateway sits at the entry point** — handles routing, rate limiting, and request validation
- Does not contain business logic or domain logic
- Flow: Client → API Gateway → Notification Service

### Component 2: Notification Service

- **Receives validated requests** from API Gateway
- Parses the notification request and determines the target channel (email, SMS, push)
- **Publishes messages to Kafka** by channel topic (email topic, SMS topic, push topic)
- Returns a job ID to the client immediately (synchronous response, asynchronous delivery)

### Component 3: Kafka Message Queue

- **Buffers notifications by channel** using separate topics (email, SMS, push)
- **Also handles preference change events** — single queueing system for both notification messages and user preference updates (simplified from initial design that had both Kafka and SQS)
- **Partitioned by user ID** for even load distribution and ordering guarantees per user
- Partition count scales with worker count to avoid idle consumers

### Component 4: Channel Workers

- **Each channel has dedicated workers** subscribing to their Kafka topic
- Workers check user preferences from **Redis cache** before sending
- Workers integrate with third-party SDKs (SendGrid for email, Twilio for SMS, Firebase for push)
- **Retry logic:** 5xx errors from third-party → retry with exponential backoff; 4xx errors → send directly to dead letter queue (no retry, won't fix itself)
- **Dead letter queue** for permanently failed notifications → alert worker notifies engineering team

### Component 5: User Preference Cache (Redis)

- **Event-driven cache invalidation:** When user changes preferences, event published to Kafka → worker updates Redis cache
- Workers check Redis before every send to ensure opt-out compliance
- Cache-first approach keeps latency low (no database call per notification)

### Component 6: Idempotency and Duplicate Prevention

- **Redis distributed lock** acquired per notification ID before processing
- **Lock TTL based on third-party API average response time** (30 seconds with buffer)
- After successful send, **write confirmation to DynamoDB asynchronously** (via side channel) with correlation ID from third-party provider
- Before sending, worker checks DynamoDB for existing confirmation — if found, skip
- **Chose DynamoDB over PostgreSQL** for write-heavy workload at 50K notifications/second
- **Asynchronous DynamoDB writes** (not synchronous) to reduce latency and cost — identified during session that synchronous writes at 50K/sec would be expensive and add unnecessary latency

### Component 7: Logging and Observability

- **Synchronous write to local file** (fast, no network call)
- **EFK stack** (Elasticsearch, Filebeat, Kibana) ships logs asynchronously from local files to centralized search
- Log at entry (notification received) and exit (sent successfully / failed / dead letter)
- Provides full audit trail without impacting send latency

### Component 8: Third-Party Failover

- **Config-driven provider selection** — workers read configuration to determine which provider to use per channel
- If primary provider goes down (e.g., SendGrid), update config to route to backup provider (e.g., Mailgun)
- Avoids hard-coding provider logic in worker code
- Manageable for a 4-person team — no complex failover orchestration

---

## Risks Identified

1. **Worker crash before logging** — If a worker crashes at the very first step (before any database or file log), there's no record of the notification being processed. Kafka redelivers the message, but debugging is harder without entry logs. Mitigate with entry-point logging to local file (fast, synchronous) before any processing begins.

2. **DynamoDB eventual consistency** — Brief window where two workers could both miss each other's write and send duplicates. Mitigate with Redis distributed lock as the primary deduplication layer; DynamoDB serves as secondary confirmation.

3. **Redis outage** — If Redis goes down, preference checks fail and distributed locks are unavailable. Workers could send to opted-out users or create duplicates. Mitigate with circuit breaker pattern and fallback to database preference lookup (slower but correct).

4. **Third-party rate limits** — SendGrid/Twilio impose rate limits. At 50K notifications/second, workers could hit provider rate limits. Mitigate by monitoring 429 responses from providers and implementing backoff; distribute sends across multiple provider accounts if needed.

5. **Kafka partition rebalancing** — Adding partitions to scale workers triggers consumer group rebalancing, which temporarily pauses consumption. Mitigate by over-provisioning partitions initially and scaling workers within existing partition count.

---

## Scoring Feedback

### What Went Well

- **Clean async pipeline design** — synchronous API submission with asynchronous delivery is the correct pattern for this scale. Showed good instinct from the start.
- **Pragmatic technology choices** — managed services for sending (SendGrid, Twilio), Kafka for buffering, Redis for caching and locks, DynamoDB for write-heavy storage. Each tool fits its role.
- **Idempotency thinking** — identified the duplicate send problem (worker crashes after send but before offset commit), proposed correlation ID + DynamoDB log + distributed lock. This is Staff-level operational thinking.
- **Real-time cost optimization** — caught the DynamoDB synchronous write cost problem mid-session and revised to asynchronous writes. This shows ability to evaluate and revise decisions under pressure.
- **Simplified architecture mid-design** — initially proposed Kafka + SQS for different concerns, then consolidated to Kafka-only when pushed. Shows willingness to simplify rather than over-engineer.
- **Partitioning strategy** — initially considered region-based partitioning, then simplified to user ID partitioning when the complexity wasn't justified. Good self-correction.

### What Needs Work

1. **Lead with tradeoffs, don't wait to be pushed** — the DynamoDB cost insight came reactively (after scoring). In an interview, surface cost-performance tradeoffs proactively: "I could write synchronously to DynamoDB for safety, but at 50K writes/second that's expensive. Instead, I'll write asynchronously after confirmation and use distributed locks as the primary dedup layer."
2. **Lock timeout reasoning was initially weak** — proposed 1-hour TTL initially, which would violate the SLA. Should immediately connect lock TTL to third-party API response time + buffer. Got there eventually, but took coaching.
3. **User preferences placement** — didn't initially design where preference checking fits in the pipeline. When asked, placed it at the worker level (correct), but should have included it in the initial architecture sketch.
4. **Communication clarity** — some explanations required multiple attempts to convey clearly (e.g., the EFK logging discussion, region vs user ID partitioning). Practice delivering component descriptions in 2-3 clean sentences.

### Key Gap for Staff Level

> Staff level requires **proactive cost-benefit analysis** before being asked. Example: "For idempotency tracking at 50K notifications/second, I have three options: (1) synchronous DynamoDB write before send — safest but adds 5-10ms latency per notification and costs ~$X/month in write units; (2) asynchronous write after send with distributed lock — slightly less safe but much cheaper and faster; (3) Redis-only with TTL — cheapest but loses data if Redis restarts. Given our SLA and team size, option 2 gives us the best cost-reliability balance."

---

## Key Learnings from This Session

### Notification System Design Pattern

| Component | Technology | Role |
|-----------|-----------|------|
| API Layer | REST API via API Gateway | Accept notifications, return job ID |
| Orchestration | Notification Service | Parse, route to correct channel |
| Buffering | Kafka (partitioned by user ID) | Decouple submission from delivery |
| Delivery | Channel workers + third-party SDKs | Actually send messages |
| Preferences | Redis cache + Kafka events | User opt-in/opt-out compliance |
| Idempotency | Redis lock + DynamoDB async write | Prevent duplicate sends |
| Logging | Local file + EFK stack | Audit trail without latency impact |
| Failover | Config-driven provider selection | Switch providers without code change |

### Sync vs Async Decision Points

| What | Sync or Async | Why |
|------|--------------|-----|
| Client API call | Sync | Simple, returns job ID immediately |
| Notification delivery | Async | Decoupled, scalable, handles retries |
| DynamoDB confirmation write | Async | Reduces latency and cost at scale |
| Log write to file | Sync | Ensures audit trail even if worker crashes |
| Log shipping to Elasticsearch | Async | EFK handles aggregation in background |

### Error Handling by HTTP Status

| Status Code | Action | Reason |
|-------------|--------|--------|
| 2xx | Log success, commit offset | Delivered successfully |
| 4xx | Dead letter queue, no retry | Client error, won't fix itself |
| 429 | Retry with backoff | Rate limited by provider |
| 5xx | Retry with exponential backoff | Server error, likely transient |

---

## Recommendations for Session 4

- Pick a **monitoring/observability platform** or **data pipeline** — continues the stretch topic progression
- Focus on **proactive tradeoff articulation** — present 2-3 options with cost-benefit analysis before the interviewer asks
- Practice **explaining distributed locking** more fluently — lock acquisition, TTL reasoning, heartbeat vs fixed timeout
- Tighten delivery — aim for 2-3 sentence component descriptions, no trailing off
- Time yourself: 5 min clarify, 3 min constraints, 10 min architecture, 5 min deep dive, 5 min scaling/tradeoffs

---

## Progress Tracking

| Session | Topic | Score | Key Improvement |
|---------|-------|-------|-----------------|
| Week 2, Session 1 | Internal Developer Platform | 6.5 / 10 | Baseline — identified framework gaps |
| Week 2, Session 2 | Rate Limiting System | 7.5 / 10 | +1.0 — constraint discipline, bottleneck thinking |
| Week 3, Session 3 | Notification System | 7.8 / 10 | +0.3 — idempotency depth, real-time cost optimization |
