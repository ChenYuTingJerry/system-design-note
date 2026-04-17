# Rate Limiter — System Design Practice Session

**Date:** 2026-04-17
**Variant:** API Gateway rate limiter
**Mode:** Step-by-step with Claude as interviewer coach

---

## Step 1: Clarify Requirements

### Functional Requirements — Questions Asked

#### Q1: Is the rate limiter based on configurable rules?
- **My instinct:** ✅ Good question — probes the policy model
- **Why it matters:** Fixed policy vs configurable rules → completely different design (configurable needs a control plane: rule storage, distribution, hot-reload, versioning)
- **Follow-ups I should have asked:**
  - Granularity of a rule? (per user / API key / IP / endpoint / tenant)
  - What dimensions? (req/sec, bandwidth, concurrent connections, cost-weighted)
  - How dynamic? (static config / hot-reload / real-time UI updates)
  - Who defines them? (platform / service owners / customers)
- **Decision for this session:**
  - Configurable rules with hot-reload
  - Granularity: per user + per endpoint
  - Dimension: requests per second
  - Tiered: free vs paid

#### Q2: What's the granularity of a rule?
- **My instinct:** ✅ Right follow-up
- **Why it matters:** Granularity determines counter cardinality → memory cost, hot-key risk, consistency model
  - Per-user only: ~10M counters
  - Per-user + endpoint: ~500M counters
  - + region: billions
- **Layers I missed:**
  - **Per-IP** for unauthenticated traffic (login, public APIs)
  - **Per-API-key** for B2B / programmatic access (different from user)
  - **Global per-endpoint** as downstream circuit breaker
- **Real gateways apply multiple limits in sequence** (per-IP AND per-user AND per-endpoint-global)
- **Decision for this session:**
  - Per-user + per-endpoint (primary)
  - Per-IP (fallback for unauth)
  - Global per-endpoint (circuit breaker)

#### Q3: What's the behavior when a request is rate-limited?
- **My instinct:** ✅ Got the most common answer (HTTP 429)
- **Why it matters:** Product decision disguised as technical — affects client UX, retry behavior, and whether you need extra infra (queue, cache for degraded responses)
- **The 4 main behaviors:**
  1. **Reject (HTTP 429)** — most common, simple, pushes burden to client. Used by GitHub, Twitter, most public APIs.
  2. **Queue / Wait** — server holds request until quota frees. Better UX but adds latency, eats memory, bad fit for HTTP.
  3. **Degrade / Serve Cached** — return stale/partial response. Great for read-heavy user-facing (Uber surge cache, Netflix fallback). Needs cache layer.
  4. **Shadow Mode** — log-only, no enforcement. Critical for safe rollout of new rules. Usually a per-rule feature flag.
- **Critical headers to return on 429:**
  - `Retry-After: 30`
  - `X-RateLimit-Limit: 100`
  - `X-RateLimit-Remaining: 0`
  - `X-RateLimit-Reset: <unix_ts>`
- **Staff-level point to make in interview:** "Behavior should be per-rule configurable — some reject, some degrade, some shadow. Always return informative headers so clients can do intelligent backoff."
- **Decision for this session:**
  - Default: Reject with 429 + headers
  - Support Shadow Mode as per-rule flag
  - Queue/degrade deferred to Step 5 alternatives

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

---

## Speaking Tips
- Status codes are spoken as digits: "four-twenty-nine" or "four-two-nine", never "four two nine". Same for 200, 500, etc.

---

## Connections to My Stack
- **Configurable rules with hot-reload** ≈ Argo Workflows `WorkflowTemplate` CRDs — declarative, versioned, controller watches for changes
