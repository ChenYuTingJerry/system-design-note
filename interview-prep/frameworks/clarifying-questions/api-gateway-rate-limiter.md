# Clarifying Questions Checklist — API Gateway Rate Limiter

A reusable checklist for any "design a rate limiter" or "design an API gateway with rate limiting" question.

---

## Functional Requirements

### Policy Model
- [ ] Is the policy fixed or **configurable** via rules?
- [ ] Static config file or **hot-reload** from a control plane?
- [ ] Who owns rule definitions? (platform / service owners / customers self-serve)
- [ ] Versioning and rollback for rule changes?

### Granularity
- [ ] Per **user** / per **API key** / per **IP** / per **tenant**?
- [ ] Per **endpoint** or global?
- [ ] **Combinations** (e.g., user × endpoint)?
- [ ] Per **region** for multi-region deployments?
- [ ] **Tiered** limits (free vs paid, internal vs external)?

### Limit Dimensions
- [ ] Requests per second / minute / hour?
- [ ] Bandwidth (bytes/sec)?
- [ ] Concurrent connections?
- [ ] Cost-weighted (some endpoints "cost" more)?

### Behavior When Limited
- [ ] **Reject** with HTTP 429? (most common)
- [ ] **Queue** the request and serve when quota frees up?
- [ ] **Degrade** (serve cached / partial response)?
- [ ] **Shadow mode** (log only, don't block — for rolling out new rules)?
- [ ] Response headers? (`Retry-After`, `X-RateLimit-Limit`, `X-RateLimit-Remaining`, `X-RateLimit-Reset`)

### Edge Cases
- [ ] Allowlist / denylist support?
- [ ] Burst handling — should clients be allowed short spikes above the limit?
- [ ] What about retries — does the client's retry storm count against them?
- [ ] Authenticated vs unauthenticated traffic — different limits?

---

## Non-Functional Requirements

### Scale
- [ ] Requests per second (peak vs average)?
- [ ] Number of unique rate-limit keys (users × endpoints)?
- [ ] Number of rules in the system?

### Latency
- [ ] Latency budget added per request? (typical: <5ms p99)
- [ ] Where does the rate limiter sit? (gateway, sidecar, library)

### Availability
- [ ] SLO? (99.99%?)
- [ ] **Fail-open or fail-closed** if the rate-limit store is down? (huge trade-off)

### Consistency
- [ ] Strict global counting (every node sees the same count) — slow but accurate?
- [ ] Eventually consistent (each node has local view) — fast but allows over-limit?
- [ ] Acceptable over-shoot percentage?

### Deployment
- [ ] Single region or multi-region?
- [ ] How to handle cross-region rate limiting? (global vs per-region quotas)

---

## Common Traps to Avoid
1. **Jumping to "use Redis with token bucket"** before clarifying granularity and consistency requirements
2. **Forgetting the control plane** — how do rules get distributed to gateway nodes?
3. **Ignoring fail-open vs fail-closed** — this is a Staff-level decision
4. **Not asking about multi-region** — changes everything if it comes up later in deep-dive
