# Rate Limiter — System Design Practice Session

**Date:** 2026-04-17
**Variant:** API Gateway rate limiter
**Mode:** Step-by-step with Claude as interviewer coach

---

## Step 1: Clarify Requirements

### Functional Requirements — Questions Asked

#### Q1: Are the rate limiter rules fixed or configurable?
- **My instinct:** ✅ Good first question — probes the policy model
- **Why it matters:** Fixed vs configurable changes the whole architecture. Configurable requires a control plane (storage, distribution, hot-reload).
- **Decision for this session:**
  - Configurable rules
  - Hot-reload supported
  - Platform team owns rule definitions

#### Q2: What's the granularity of rate limiting?
- **My instinct:** ✅ Strong — covered all main dimensions (user, API key, endpoint, combinations, region)
- **Why it matters:** Granularity drives counter cardinality → memory cost, sharding strategy, hot-key risk
- **The dimensions to know:**
  - Per user — most common
  - Per API key — for B2B / programmatic access
  - Per IP — fallback for unauthenticated traffic
  - Per endpoint — per-route limits
  - Per region — for multi-region deployments
  - Combinations (e.g., user × endpoint)
- **Decision for this session:**
  - Primary: `(user_id, endpoint)`
  - Fallback: `(ip_address, endpoint)` for unauthenticated requests
  - Multi-region deferred to later

#### Q3: What's the behavior when a request is rate-limited?
- **My instinct:** ✅ Standard follow-up after defining inputs — asks about client-facing behavior
- **Why it matters:** This is a product decision. Affects client UX, retry patterns, and what infra you need (queue? cache?)
- **The 4 common behaviors:**
  1. **Reject with HTTP 429** — most common, simplest
  2. **Queue & serve later** — better UX but adds latency, memory
  3. **Degrade** (cached/partial response) — good for read-heavy user-facing APIs, needs cache
  4. **Shadow mode** (log-only) — safe rollout of new rules
- **Decision for this session:**
  - Reject with HTTP 429 (Too Many Requests)
  - Return `Retry-After` header
  - Also return `X-RateLimit-Limit`, `X-RateLimit-Remaining`, `X-RateLimit-Reset` for smart client backoff

---

### Non-Functional Requirements — Questions Asked

#### Q4: What's the scale (DAU and peak RPS)?
- **My instinct:** ✅ Good — scale is the right first NFR question, drives downstream decisions
- **Why it matters:** Scale determines memory cost, sharding strategy, consistency model
- **What to ask for (both numbers, not just one):**
  - **DAU** (daily active users) → drives counter cardinality / storage
  - **Peak RPS** → drives throughput / node count
- **Improvement for next time:** Ask both together — "How many DAU, and what's the peak RPS?"
- **Decision for this session:**
  - 100M DAU
  - 1M RPS peak

#### Q5: What's the latency budget?
- **My instinct:** ✅ Critical question — rate limiter is on the critical path of every API request
- **Why it matters:** Latency budget directly shapes architecture
  - < 1 ms → in-memory only, sloppy consistency
  - 1–5 ms → co-located Redis, single round trip (standard)
  - > 20 ms → unacceptable for a hot-path component
- **Tail latency > average:** at 1M RPS, even p99.9 of 50ms = 1000 slow req/sec
- **Decision for this session:**
  - p99 < 5 ms (rate-limit check alone)
  - p99.9 < 10 ms
  - This is added latency on top of normal processing
- **Architectural implications:**
  - No synchronous cross-region coordination
  - Must use co-located cache (same-AZ Redis, or in-memory)
  - May need to accept consistency trade-offs to stay fast
- **Staff-level interview move:** "At 1M RPS, even p99.9 of 50ms = 1000 slow req/sec. That's why tail latency matters more than average for rate limiters."

#### Q6: Fail-open or fail-closed when the backend store is down?
- **My instinct:** 🎯 Staff-level signal — asking this unprompted shows you think about failure modes
- **Why it matters:** Rate limiter is on the critical path. Its failure mode affects product availability and safety.
- **The core trade-off:**

| Mode | If rate limiter dies... | Pros | Cons |
|---|---|---|---|
| Fail-open | Allow all traffic | API stays available | Backend can be overwhelmed |
| Fail-closed | Reject all traffic | Backend stays safe | Rate limiter outage = full API outage |

- **Real-world pattern:** fail-open for general traffic (GitHub, Stripe), fail-closed for security-critical paths (login, payments)
- **Stack connection:** Same as K8s admission webhooks — `failurePolicy: Ignore` (fail-open) vs `failurePolicy: Fail` (fail-closed)
- **Decision for this session:**
  - Default: fail-open
  - Per-rule override: critical rules can fail-closed
  - Availability SLO: 99.99% for rate-limit check

---

### Functional Requirements — Still TODO
- [ ] Behavior when limited (reject vs queue vs degrade)
- [ ] Response format (status code, headers like `Retry-After`, `X-RateLimit-Remaining`)
- [ ] Allowlist / denylist support?
- [ ] Burst handling (token bucket vs leaky bucket implications)

### Non-Functional Requirements — Still TODO
- [ ] Scale (RPS, # users, # rules)
- [ ] Latency budget (added per request)
- [ ] Availability SLO
- [ ] Consistency model (strict global vs eventually consistent)
- [ ] Multi-region?

---

## Step 2: Back-of-envelope Estimation
*(not started)*

## Step 3: High-level Design
*(not started)*

## Step 4: Deep Dive
*(not started)*

## Step 5: Trade-offs and Alternatives
*(not started)*

---

## English Phrases I Got Corrected 💬

| Awkward | Natural |
|---------|---------|
| "Is the radiometer based on configurable rules?" | "Should the rate limiter support configurable rules, or is it a fixed policy?" |
| "What's the granular granularity of a rule? It's it per user or per endpoints." | "What's the granularity of a rule? Is it per user, per endpoint, or a combination?" |
| "What is the behavior when limited happened?" | "What's the expected behavior when a request is rate-limited?" |
| "I have seen the behavior when the request is rare rate limited, is the server respond response this the HTTP status four two nine. alias too many requests." | "The behavior I've seen when a request is rate-limited is the server responds with HTTP 429 — also known as 'Too Many Requests'." |
| "What is the latency we accept" | "What's the latency budget we need to meet?" |
| "Is the granularity per user or per API key or per endpoint. Or it's a combination? Like user and endpoint. Or per region for multi region deployments." | "What's the granularity — per user, per API key, per endpoint, or a combination? And do we need per-region granularity for multi-region deployments?" |
| "Is the right diameter rules Is the rules of rate limiter fixed or configurable?" | "Are the rate limiter rules fixed or configurable?" |
| "What is the behavior when requesters are limited." | "What's the expected behavior when a request is rate-limited?" |
| (none — "How many users per day?" is natural) | ✅ |
| "Is the red limiter felt open or fell closed if it if it store if it is done. Or break," | "Is the rate limiter fail-open or fail-closed if the backend store goes down?" |

---

## Speaking Tips
- Status codes are spoken as digits: "four-twenty-nine" or "four-two-nine", never "four two nine". Same for 200, 500, etc.
- **"budget"** is the standard interview term for NFR constraints: "latency budget", "error budget", "memory budget". Using it signals seniority.
- **fail-open** = allow traffic when system fails (prioritize availability)
- **fail-closed** = block traffic when system fails (prioritize safety)
- Universal pattern — networking, security, admission control

---

## Connections to My Stack
- **Configurable rules with hot-reload** ≈ Argo Workflows `WorkflowTemplate` CRDs — declarative, versioned, controller watches for changes

---

## 🛑 Session Pause — 2026-04-18

### Status
- ✅ Step 1 (Clarify Requirements) — COMPLETE
  - ✅ 1A Functional Requirements (Q1–Q3)
  - ✅ 1B Non-Functional Requirements (Q4–Q6)
- ⏳ Step 2 (Back-of-envelope Estimation) — NOT STARTED

### Key decisions locked in
- Configurable rules with hot-reload
- Granularity: `(user_id, endpoint)` + IP fallback
- Behavior: HTTP 429 + Retry-After headers
- Scale: 100M DAU, 1M RPS peak
- Latency: p99 < 5ms
- Failure mode: fail-open default, per-rule override

### To resume
Say to Claude: "Continue rate limiter session from Step 2."

### Open questions / items flagged for later
- Multi-region (deferred from Q2)
- Burst handling — not yet discussed
- Control plane design for rule distribution — Step 3+
- Security / auth bypass handling — not yet discussed
