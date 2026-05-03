# 8-Week Interview Prep Plan v2

**Last updated:** 2026-04-22
**Target roles:** Staff / Senior+ / Senior Platform Engineer, SRE, DevOps/Infrastructure
**Target markets:** NL + DE + IE + ES + PT + Remote-EU
**Application ratio per 10:** 2 Staff + 5 Senior+/Lead + 2 Senior + 1 flexible
**Visa:** HSM eligible (NL), Critical Skills Employment Permit (IE)

## Application Level Strategy

Original strategy: Staff-only at 5 Amsterdam companies.
Updated strategy: Mixed levels across broader EU market.

Reason: 5 Staff cold applications rejected (Uber x2, Databricks, JetBrains, Nebius). Non-EU location + no local network makes Staff-level written screening too difficult to pass consistently. Senior+ interviews are a subset of Staff prep, so preparing at Staff level still covers Senior+ interviews.

## Three Signature Stories (two delivery modes each)

### Story 1: 17LIVE Strangler Fig Migration
- **Staff mode:** Lead with ownership gap framing. "No engineer had architectural ownership. I believed that fell to me as Staff." Cross-org influence, VP alignment, decision framework.
- **Senior+ mode:** Lead with technical execution. "I designed and led a monolith-to-microservices migration using strangler fig, hexagonal architecture, and gRPC." Hands-on delivery, incremental validation, API gateway routing.
- **SRE mode:** Lead with operational constraints. "700K DAU, peak Asia-evening windows, zero tolerance for availability risk during migration." Production safety, rollback strategy, validation approach.

### Story 2: Symphox Platform Migration (Kubernetes + unified CI/CD)
- **Staff mode:** Influence without authority across teams.
- **Senior+ mode:** Built unified CI/CD, multi-cluster Kubernetes, technical execution.
- **Status:** Not yet scripted.

### Story 3: Billing Team EM Takeover
- **Staff mode:** Managing dysfunction, team turnaround, organizational problem-solving.
- **Senior+ mode:** Took over struggling team, delivered on schedule, zero attrition.
- **Status:** Not yet scripted.

## Coding Prep Plan

**Language:** Python (primary)
**Topic bank:** NeetCode 150 ([neetcode.io](http://neetcode.io))
**Daily target:** 1 Medium per day
**Benchmark:** Medium stable in 45 minutes

### Weekly Progression

| Week | NeetCode Patterns | Goal |
|------|-------------------|------|
| 3–4 | Arrays & Hashing, Two Pointers, Sliding Window, Stack | Pattern recognition + explain reasoning |
| 5–6 | Binary Search, Linked List, Trees, Heap / Priority Queue | Stable Medium in 30 min |
| 7–8 | Graphs, DP (1D only) | 45-min timed mock with English narration |

### Daily Practice Rule

After every problem, answer these three questions:
1. What pattern is this?
2. What is the time / space complexity?
3. If the interviewer asks for a better solution — what do I say?

### Week 7–8 Mock Format

- 45-minute timer, one problem
- Narrate reasoning in English throughout
- After solving: explain time/space complexity out loud

## Fluency Drills (daily practice, integrated into every session)

### Drill 1: Own the ideas, not the words
Do not memorize exact sentences. Before practicing a constraint or story, close notes and explain the core idea in your own words. If you can explain it conversationally, you own it. If you need to check notes, you don't own it yet.

### Drill 2: Stop-and-restart technique
When you catch yourself repeating a phrase or looping mid-sentence, stop immediately. Take one breath. Restart that sentence cleanly from the beginning. Do not push through the loop. This is the single most effective technique for breaking the looping habit.

### Drill 3: Practice when fresh
Do constraint and story delivery practice in the morning when energy is high. Short sessions (15-20 minutes) are better than marathon sessions. Fatigue causes looping. Fresh energy prevents it.

### Drill 4: Slow down
Speak slower than feels natural. Slower speech gives your brain time to find the next word without looping. If you feel like you're speaking too slowly, you're probably at the right speed.

## Weekly Plan

### Week 1 (Apr 20-26) — Foundation: Decision Framework + 17LIVE Story

**Completed:**
- Session 1 (Apr 20): Diagnostic. Four probes. Key gap identified: jumps to solutions without naming constraints.
- Session 2 (Apr 21): Layer 1 (Constraints) locked. Five constraints for 17LIVE. Ownership-gap framing established.
- Session 3 (Apr 22): Layers 2, 3, 4 complete. Full 4-layer story delivered. Constraint 5 factual correction (SRE didn't own service mesh). Looping pattern identified and stop-and-restart technique introduced.

**Remaining:**
- Session 4 (Apr 23): Cold delivery of five constraints. Full 4-layer story clean take. Build Senior+ delivery mode for 17LIVE story. Start 60-second self-introduction.
- Session 5 (Apr 24-26): Polish self-introduction with ownership framing. Common questions: "why leave last job", "EM to IC transition".

### Week 2 (Apr 27 - May 3) — Stories 2 and 3 + Common Questions

- Symphox platform migration story: CARL framework, Staff + Senior+ modes
- Billing team EM takeover story: CARL framework, Staff + Senior+ modes
- "Why relocate" flexible answer: NL version, DE version, IE version, ES/PT version
- "Tell me about yourself": 90-second version incorporating ownership framing
- Daily fluency drills: 15-20 min morning practice on constraints and stories

### Week 3 (May 4-10) — Role-Type Prep: Platform Engineer

- Platform engineering system design topics:
  - Kubernetes multi-cluster architecture
  - CI/CD pipeline design
  - Developer platform strategy / Internal Developer Platform (IDP)
  - GitOps patterns and tradeoffs
- Practice delivering system design answers using the decision framework (Constraints → Alternatives → Tradeoffs)
- First paid mock interview (Senior+ level recommended since 70% of applications)
- Daily fluency drills continue
- Coding: Start NeetCode 150 — Arrays & Hashing first. 1 Medium/day. After each problem: name the pattern, state complexity, prepare follow-up answer.

### Week 4 (May 11-17) — Role-Type Prep: SRE + Infrastructure

- SRE-specific topics:
  - Observability pipelines (OpenTelemetry)
  - Deployment strategies (blue-green, canary, rolling)
  - Incident response frameworks
  - On-call philosophy and operational maturity
  - Runbook design and automation
- Reframe 17LIVE and Symphox stories for SRE interviews (operational constraints, production safety, reliability)
- MongoDB Dublin SRE prep if application is active (Deployments 75% fit, SLS 70%, Observability 65%)
- Daily fluency drills continue
- Coding: Continue NeetCode 150 — Two Pointers, Sliding Window, Stack. 1 Medium/day.

### Week 5 (May 18-24) — Mock Interviews + Feedback

- Second mock interview (different role type from Week 3)
- Review and refine all three stories based on mock feedback
- Practice "probe resistance": answering follow-up questions without overclaiming or backpedaling
- Company-specific research for any active interview pipelines (1-2 hours per company, only when interview is scheduled)
- Daily fluency drills continue
- Coding: Binary Search, Linked List (NeetCode 150). 1 Medium/day.

### Week 6 (May 25-31) — System Design Deep Practice

- Two full system design practice rounds (45 min each)
- Topics: monolith decomposition, observability at scale, platform self-service design
- Practice leading with constraints before jumping to solutions (Session 1 diagnostic gap)
- Refine Senior+ vs Staff delivery for system design answers
- Daily fluency drills continue
- Coding: Trees, Heap / Priority Queue (NeetCode 150). 1 Medium/day.

### Week 7 (Jun 1-7) — Behavioral Deep Practice + Salary

- Full behavioral mock (3-4 questions, mixed Staff and Senior+ level)
- Salary negotiation scripts for different markets:
  - NL: redirect to Amsterdam market rate, never disclose Taiwan compensation
  - DE: similar approach, adjusted for German market expectations
  - IE: Critical Skills Employment Permit context, Irish market rates
- Practice "what's your expected compensation" redirects
- Refine stories based on all feedback received
- Daily fluency drills continue
- Coding: Graphs + timed mock sessions begin (45 min, English narration throughout).

### Week 8 (Jun 8-14) — Final Polish + Active Interview Support

- Final mock interview (full loop simulation if possible)
- Company-specific prep for any scheduled interviews
- Review all three stories one final time: deliver cold, clean, one take each
- Confidence building: full 4-layer 17LIVE story in under 3 minutes
- All common questions delivered cleanly without notes
- Coding: DP (1D only), continue timed mocks.

## Decision Framework (internalized)

**Layer 1 — Constraints:** What was non-negotiable about the situation
**Layer 2 — Alternatives:** What approaches could have solved the problem in principle
**Layer 3 — Tradeoffs:** What each alternative optimizes for and sacrifices
**Layer 4 — Counterfactuals:** What would have to change for me to pick a different alternative

Decision framework (thinking structure) is not the same as delivery framework (communication structure). Both needed. Decision framework lives in your head. Delivery framework lives in the conversation.

## Staff-level Filler Phrases (internalized)

1. "The constraint that mattered most here was..."
2. "What made this a hard constraint rather than a preference was that [consequence] couldn't be recovered after the fact."
3. "Our [operational metric] was [number], on a platform reaching [larger metric]. The constraint that mattered for [decision] was [the operational one], because [why]."
4. "Leadership had made a deliberate tradeoff here — they chose to prioritize [X] over [Y], which meant that any architectural approach requiring [Z] was off the table."
5. "The technical foundation gave us [X, Y, Z] for free — those were already production-proven. What was missing was [specific layer]. So our architectural scope was bounded to [the delta], not the full system."
6. "I prioritized the work with the highest [coupling fan-out / blast radius / cross-team impact], because doing that first produced broader benefit per unit of effort."
7. "At [company], I observed that no engineer had taken architectural ownership of [specific problem]. I believed that was a responsibility that fell to me as a Staff engineer — so I proactively [what I did]."

## Key Principles (do not drift)

- **Accuracy is non-negotiable.** No unconfirmed tools, metrics, or numbers in any material.
- **Every reframing, verify line by line.** Catch overclaims in coaching, not in interviews.
- **Never disclose Taiwan compensation.** Redirect to target market expectations.
- **Two delivery modes per story.** Staff mode (influence, ownership) and Senior+ mode (execution, technical depth). Same facts, different emphasis.
- **Role-type prep over company-specific prep.** Research specific companies only when interview is scheduled.
- **Fluency comes from owning ideas, not memorizing words.** Stop-and-restart technique for looping. Practice when fresh. Slow down.
- **Coding:** Explain the pattern and complexity out loud, not just solve the problem.

## Success Metrics

### Coding milestones
- End of Week 4: Arrays, Two Pointers, Sliding Window, Stack patterns solid. Can explain approach in English.
- End of Week 6: Medium problems stable in 30 minutes. Complexity analysis automatic.
- End of Week 8: 45-minute timed mock format comfortable. English narration fluent.
