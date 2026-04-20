# Diagnostic Session — Part 1 (Probes 1 & 2)

**Date:** 2026-04-20
**Coach:** Claude
**Format:** Voice-based diagnostic, English
**Coverage:** System design decision-making, behavioral storytelling (influence without authority)

## Session Context

First diagnostic session with coaching setup: medium pushback calibration, honest read (no encouragement mode), notes committed to existing interview-prep repo. Goal was to probe weakest areas before planning the 8-week arc.

## Probe 1 — System Design Decision-Making

### Scenario
Asked to walk through the 17LIVE monolith-to-microservices migration decision: specifically, what alternatives were rejected and what would have had to be true to pick a different path.

### What I demonstrated
- Good architectural choice (strangler fig) for the actual constraints
- Got organizational buy-in from supervisor and VP
- Understood production-risk constraint (millions of active users)
- Pragmatic team structure: one feature team maintained existing monolith, another team (3 engineers I led) proved new architecture on new features
- Identified organizational constraint: other senior engineers couldn't dedicate time, so I absorbed the spec reverse-engineering cost myself to unblock them
- Shipped a working proof-of-concept that de-risked broader migration

### The gap (Staff-level signal)
I operate on **instinct and pattern-matching**, not on **structured decision frameworks**. When asked counterfactual questions ("what would make you pick a different approach?"), I describe what actually happened instead of articulating the decision principles.

**Senior-level answer:** "We used strangler fig because it allows incremental migration with zero downtime."

**Staff-level answer:** "We chose strangler fig because constraint A ruled out big-bang, constraint B ruled out parallel run, and given constraint C, strangler fig was the only approach that preserved feature velocity while proving the architecture. If constraint X changed, we'd revisit."

### What I admitted honestly
I'm not thinking in decision frameworks. I'm answering by instinct. This is the real gap.

### What I need to build
A decision-making vocabulary:
- Name alternatives explicitly (big-bang, branch-by-abstraction, anti-corruption layer, parallel run, event interception)
- Articulate what each optimizes for and what it fails on
- Explain the specific constraints that made my choice right for that situation
- Flip the counterfactual: "If X were true, I'd pick Y instead"

### Next action
Take the 17LIVE story and reframe it with explicit decision frameworks. Practice articulating counterfactuals under pressure.

## Probe 2 — Behavioral Narrative (Influence Without Authority)

### Scenario
Asked for a CARL story for "influence without authority." Initially confused with a different story (billing team EM takeover), which on closer look was actually a *formal authority* story — not the right fit. Pivoted to the correct story: unified CI/CD platform migration at Symphox.

### Important correction to memory
My previous notes described the billing team conflict as "resolved without formal authority." This is inaccurate. When my supervisor asked me to take on the billing team, accepting the role gave me formal authority. That story is still valid — but it's a "took over a broken team and fixed it" story, not an "influence without authority" story.

**The real influence-without-authority story is the Symphox unified platform migration.**

### The story (CARL)

**Context:**
- Services scattered across EC2 and Lambda
- Manual deployments owned by individual developers
- Only the developer who built a service could maintain it — knowledge silos
- Teams were comfortable and didn't see this as a problem
- Their managers didn't want to invest in migration

**Action:**
- Identified this as a silo problem blocking engineer cross-support
- Tried to convince team leads directly — rejected because they didn't see the benefit
- Built a POC/MVP of the unified platform (Kubernetes + Azure DevOps + Terraform) to prove value before asking for commitment
- Invited interested team members (not managers) to participate in the migration

**Result:**
- Two teams successfully migrated to the unified platform
- Silos were fixed through common deployment process and tooling
- Platform adoption happened through demonstrated value, not top-down mandate

**Learning:**
Platform adoption isn't about pushing the newest or fanciest tech. It's about understanding the teams' existing technical stack, their constraints, and building something they can maintain and own. Tool choice must fit the problem, not the other way around.

### The gap
My storytelling is meandering. I jump between ideas, repeat myself, lose the thread mid-sentence. The story is good — a real Staff-level influence story with a real principled learning — but I can't deliver it cleanly under pressure. In a 2-minute behavioral answer, the interviewer would lose me.

At one point I dismissed my own learning by saying "it's not a good one" — but it IS good. I just couldn't articulate it cleanly in the moment.

### What I need to build
A scripted 90–120 second version of this story that hits all four CARL beats without fumbling. Practice until it's natural conversational speech.

### Next action
Script the tight version. Record myself delivering it. Iterate until it flows naturally in 2 minutes.

## Overall Diagnostic Read

**Strongest area:** Platform architecture and pragmatic problem-solving. I make good decisions and I ship.

**Weakest area:** Articulating the reasoning behind those decisions in a way that transfers to new contexts. I can describe what I did. I'm weaker at explaining *why* I'd make similar decisions on a different problem at a different company.

**The good news:** Both gaps are coachable with 4–6 weeks of structured practice.

**The hard truth:** This cannot be faked in an interview. A good Staff interviewer will surface the framework gap in 15 minutes. Real decision vocabulary and real delivery practice are required.

## Open items for Part 2 (this afternoon)
- Probe 3: Coding and platform craft
- Probe 4: English fluency under pressure
- Decision on 8-week plan once full diagnostic is complete
- Specific concern to revisit: scope of Week 5 + Week 7 take-home (Mini Observability Operator may be over-scoped for interview prep)

## Follow-up notes for coach
- Confirmed coaching tone: honest read, not encouragement
- Voice-based practice is viable, but transcription artifacts require patience
- Candidate prefers structured frameworks over generic advice
- Do not fabricate alternatives in system design stories — honest "we didn't formally consider X" is better than invented comparisons
