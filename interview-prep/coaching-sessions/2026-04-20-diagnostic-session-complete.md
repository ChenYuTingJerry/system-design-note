# Diagnostic Session — Complete (All 4 Probes)

**Date:** 2026-04-20
**Coach:** Claude
**Format:** Voice-based diagnostic, English
**Duration:** ~3 hours (morning + afternoon)
**Coverage:** System design, behavioral storytelling, coding/platform craft, English fluency under pressure

## Session Context

First diagnostic session. Coaching contract: medium pushback calibration, honest read (no encouragement mode). Goal: probe weakest areas before committing to an 8-week plan.

## Probe 1 — System Design Decision-Making

### Scenario
Walk through 17LIVE monolith-to-microservices migration: which alternatives were rejected, what would have had to be true to pick a different path.

### What I demonstrated
- Good architectural choice (strangler fig) for the actual constraints
- Got organizational buy-in from supervisor and VP
- Understood production-risk constraint (millions of active users)
- Pragmatic team structure: one feature team maintained existing monolith, another team (3 engineers I led) proved new architecture on new features
- Absorbed organizational constraint myself: other senior engineers couldn't dedicate time, so I did the spec reverse-engineering to unblock them
- Shipped working proof-of-concept that de-risked broader migration

### The gap (Staff-level signal)
I operate on instinct and pattern-matching, not on structured decision frameworks. When asked counterfactual questions, I describe what actually happened instead of articulating decision principles.

Senior-level answer: "We used strangler fig because it allows incremental migration with zero downtime."

Staff-level answer: "We chose strangler fig because constraint A ruled out big-bang, constraint B ruled out parallel run, and given constraint C, strangler fig was the only approach that preserved feature velocity while proving the architecture. If constraint X changed, we'd revisit."

### What I admitted honestly
I'm not thinking in decision frameworks. I'm answering by instinct.

### What I need to build
Decision-making vocabulary:
- Name alternatives explicitly (big-bang, branch-by-abstraction, anti-corruption layer, parallel run, event interception)
- Articulate what each optimizes for and what it fails on
- Explain constraints that made my choice right for that situation
- Flip counterfactuals: "If X were true, I'd pick Y instead"

## Probe 2 — Behavioral Narrative (Influence Without Authority)

### Important correction to memory
Previous notes described the billing team conflict as "resolved without formal authority." This is inaccurate. When my supervisor asked me to take on the billing team, accepting the role gave me formal authority. That story is still valid but it's a "took over a broken team and fixed it" story, not an "influence without authority" story.

**The real influence-without-authority story is the Symphox unified platform migration.**

### The Symphox story (CARL)

**Context:**
- Services scattered across EC2 and Lambda
- Manual deployments owned by individual developers
- Only the developer who built a service could maintain it — knowledge silos
- Teams were comfortable, didn't see this as a problem
- Their managers didn't want to invest in migration

**Action:**
- Identified silo problem blocking engineer cross-support
- Tried to convince team leads directly — rejected, they didn't see benefit
- Built POC/MVP of unified platform (Kubernetes + Azure DevOps + Terraform)
- Invited interested team members (not managers) to participate in migration

**Result:**
- Two teams successfully migrated to unified platform
- Silos fixed through common deployment process and tooling
- Adoption happened through demonstrated value, not top-down mandate

**Learning:**
Platform adoption isn't about pushing the newest or fanciest tech. It's about understanding teams' existing technical stack, their constraints, and building something they can maintain and own. Tool choice must fit the problem, not the other way around.

### The gap
Storytelling meanders. Jumping between ideas, repeating, losing the thread. The story is good — a real Staff-level influence story with a principled learning — but delivery breaks down under pressure. At one point I dismissed my own learning by saying "it's not a good one" — but it IS good. Just inarticulate in the moment.

### What I need to build
Scripted 90–120 second version of this story that hits all four CARL beats without fumbling. Practice until it's natural conversational speech.

### Three signature stories for three different questions
1. **Strangler fig (17LIVE)** → system design decision-making
2. **Platform adoption (Symphox)** → influence without authority
3. **Billing team (EM role)** → team leadership and career development

## Probe 3 — Coding and Platform Craft

### Scenario
Inherited service at Uber with P99 latency of 2s (target 200ms). Walk through first 30 minutes of investigation.

### What I demonstrated
- Systematic starting point: metrics first, then recent changes, then drill into DB or service
- Correctly prioritized distributed traces for tail latency in microservices
- Prioritized by impact: DB query optimization (80% of latency) before cache
- First hypothesis on slow query: full table scan / missing index
- Critically: asked "why was the index missing?" — root cause thinking, not just symptom fixing
- Framed the root cause as a process gap (was index forgotten when field was added?)

### The gap
Thinking is solid. Delivery is hesitant — I describe my reasoning like I'm unsure, when I'm actually correct. At Uber, a Staff engineer states hypotheses with confidence: "Full table scan is the first hypothesis because..." I say it like a question.

### What I need to build
Confidence in delivery. State hypotheses directly. Use language like "I'd first check X because Y" rather than "I would maybe check X, I think."

## Probe 4 — English Fluency Under Pressure

### Scenario
60-second opening framework for "design a global distributed cache with sub-100ms reads from any region while ensuring consistency."

### What I demonstrated
- Vocabulary is sufficient — knew technical terms
- Could hold multi-turn technical conversation
- When pushed on consistency, made a principled tradeoff (eventual consistency for performance)

### The gap
- Skipped clarifying questions — jumped straight to solution
- Grammar slips under pressure
- Sentence structure breaks when thinking hard
- No filler phrases to buy think time
- Default to "build solution" instead of "understand constraints first"

### What I need to build
- 10–15 filler phrases to buy think time ("Let me clarify the scope first", "There are a few ways to approach this")
- Habit of asking clarifying questions BEFORE designing
- Drill openings of signature stories until first 30 seconds are automatic

### Honest verdict on English
Good enough to pass Staff interviews at most EU companies (Adyen, Booking, Mollie, Picnic — used to non-native speakers). Not polished enough to let a weak technical answer pass. If technical answer is excellent, English won't block. If technical answer is borderline, English could push it into "no hire." Fix technical articulation first; English in parallel.

## Overall Diagnostic Read

### Strongest area
Platform architecture and pragmatic problem-solving. I make good decisions and I ship.

### Weakest area
Articulating reasoning in a way that transfers to new contexts. I can describe what I did at 17LIVE or Symphox. I'm weaker at explaining *why* I'd make similar decisions on a different problem at a different company.

### Key pattern across all four probes
I'm a strong engineer operating on instinct. The gap is not thinking ability — it's converting thinking into interview-ready language.

### Good news
Both gaps (decision frameworks + delivery) are coachable with 4–6 weeks of structured practice.

### Hard truth
Cannot be faked in an interview. A good Staff interviewer will surface the framework gap in 15 minutes. Real decision vocabulary and real delivery practice are required.
