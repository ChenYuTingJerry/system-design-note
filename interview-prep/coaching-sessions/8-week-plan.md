# 8-Week Interview Prep Plan

**Start date:** 2026-04-21 (Session 2)
**Target:** Staff / Principal / Lead Platform Engineer roles in NL and Barcelona
**Weekly commitment:** 25–35 hours
**Total budget:** ~200–280 hours

## Strategic Framing

### Application strategy
Apply to both Staff and Senior roles. Dutch market has fewer Staff-level roles than US; many top-level IC roles in NL are labeled "Senior" or "Lead" but carry Staff-level scope and compensation. Title is secondary to TC and sponsorship.

### Target companies
Uber Amsterdam (primary), Preply Barcelona (Route B), Nebius Amsterdam, plus inbound via Undutchables, Relocate.me, talent.io, Wellfound.

### Prep focus hierarchy
1. Decision-framework vocabulary (primary gap)
2. Behavioral story delivery (secondary gap)
3. System design pattern fluency (weekly mocks handle this)
4. English delivery polish (parallel, not primary)

## Weeks 1–2: Foundation

### Decision-framework vocabulary (3–4 hrs/week)
- Take 17LIVE strangler fig story and reframe with explicit decision frameworks
- Practice naming alternatives (big-bang, branch-by-abstraction, parallel run, anti-corruption layer, event interception)
- Drill counterfactuals: "If X were true, I'd pick Y because Z"
- Apply framework to one system design problem: distributed cache

### Story scripting and delivery (2–3 hrs/week)
- Script tight 90–120 second CARL versions of:
  - Symphox platform migration (influence without authority)
  - Billing team EM takeover (team leadership)
  - 17LIVE strangler fig (technical decision-making)
- Record delivery, iterate until natural

### One diagnostic mock (3–4 hrs)
- Self-recorded, focused on system design
- Review for decision-framework gaps

### Targeted English work (1–2 hrs)
- Record answers to 5 common questions (60-sec intro, "why leave last job", "why NL", "EM→IC", "tell me about yourself")
- Transcribe, fix delivery issues
- Memorize 10–15 filler phrases for think-time

### Total: ~12–15 hrs/week

## Weeks 3–8: Applied Practice

### Weekly paid mocks (2–3 hrs/week per mock)
- Exponent or Hello Interview
- Alternate system design and behavioral rounds
- Staff-level coaches only

### Between-mock reinforcement (2–3 hrs/week)
- Review mock feedback
- Drill weak areas exposed by mocks
- Apply decision frameworks to new problems

### System design practice (2–3 hrs/week)
Pick 6 core problems across the 8 weeks. Go deep on each:
1. Distributed cache (start in Week 1)
2. Rate limiter
3. Message queue / Job scheduler
4. Metrics monitoring system (leverage OpenTelemetry knowledge)
5. One complex end-to-end problem (YouTube or Uber)
6. Notification system

**Do NOT** grind through 25+ generic problems. Understand patterns, trust transfer.

### LeetCode medium-to-hard (2–3 hrs/week)
Platform-relevant patterns:
- Caching (LRU, LFU)
- Queuing (sliding window, rate limiting)
- Distributed consensus concepts (simplified)
- String/array manipulation (speed practice in Python)

### Total: ~8–12 hrs/week interview-focused

## Excluded from this plan

### Mini Observability Operator (Kubebuilder project)
**Decision:** Treat as side project for skill development, NOT interview prep.

Rationale:
- 40–60 hour scope too large for interview prep ROI
- EU Staff Platform interviews rarely require take-homes of this size
- Time better spent on mocks and frameworks
- Do this project AFTER landing offer, or in parallel with prep (weekends only), labeled as learning

## Three Signature Stories

Keep these distinct and matched to question types:

1. **17LIVE strangler fig** → system design and technical decision-making questions
2. **Symphox platform migration** → influence without authority questions
3. **Billing team EM takeover** → team leadership and management questions

## Success Metrics

### End of Week 2
- Can articulate 17LIVE story with explicit alternatives, constraints, counterfactuals
- Three CARL stories scripted and delivered cleanly under 2 minutes each
- 10+ filler phrases internalized for think-time

### End of Week 4
- Two paid mocks completed with feedback reviewed
- Decision-framework language emerging naturally in mocks
- Behavioral delivery tightened (feedback from mock coaches)

### End of Week 8
- 6 paid mocks completed
- Applied to 10+ roles across NL and Barcelona
- First-round interviews scheduled or in progress
- Confidence to drive 45-minute Staff-level system design conversation

## Session Schedule

- Session 1: 2026-04-20 — Diagnostic (complete)
- Session 2: 2026-04-21 — Decision frameworks: 17LIVE strangler fig
- Session 3: TBD — Behavioral story: Symphox CARL scripting
- Subsequent sessions: TBD based on mock schedule

## Homework Before Session 2

1. Commit these notes to GitHub repo
2. Record myself answering: "Walk me through how you migrated 17LIVE from monolith to microservices, and what alternatives you considered." (5 minutes max)
3. Write down actual business constraints that made strangler fig the right choice (user scale, timeline, team size, feature velocity pressure)
4. Review the diagnostic notes — absorb where the gaps are before we start framework work
