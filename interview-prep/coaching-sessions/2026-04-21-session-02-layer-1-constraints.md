
# Session 2 — Decision Framework Layer 1: Constraints (17LIVE)

**Date:** 2026-04-21
**Coach:** Claude
**Duration:** ~90 minutes
**Focus:** Decision-framework vocabulary — Layer 1 (Constraints) applied to the 17LIVE strangler fig story
**Status:** Layer 1 complete. Layers 2 (Alternatives), 3 (Tradeoffs), and 4 (Counterfactuals) scheduled for Session 3.

## Session Context

First working session of the 8-week plan. Fresh energy, full 90 minutes, no pre-scripted homework — did the constraint-listing exercise live so Claude could push on unrehearsed reasoning in real time.

Also flagged: target companies updated mid-session to Uber, Databricks, JetBrains, ING, Adyen (replacing the earlier Uber + Preply + Nebius list in the plan doc). Plan doc needs updating separately.

## Decision Framework vs. Delivery Framework — clarified

Important distinction established this session:

- **Decision framework** = thinking structure (Constraints → Alternatives → Tradeoffs → Counterfactuals). Lives in my head. How I reason about a problem.
- **Delivery framework** = communication structure (CARL, STAR, HelloInterview 5-step). Lives in the conversation. How I structure my speech.

Both are needed. Session 2 targets the decision framework because it's the deeper gap from the Session 1 diagnostic.

## The Four Layers of Decision Framework

1. **Constraints** — what was non-negotiable about the situation
2. **Alternatives** — what approaches could have solved the problem in principle
3. **Tradeoffs** — what each alternative optimizes for and sacrifices
4. **Counterfactuals** — what would have to change for me to pick a different alternative

Order matters: you can't articulate tradeoffs without alternatives, and you can't articulate counterfactuals without constraints. Build in order.

## Layer 1 — Five Constraints (17LIVE monolith→microservices migration)

### Constraint 1 — User scale and operational load

> We had 50K to 150K DAU on a globally distributed platform serving multiple regions, with 50M registered users. This constraint had the most impact during peak Asia-evening windows. Any architectural decision that put whole-system availability at risk during those windows was off the table — an outage during peak would have been a global brand incident across all regions we served.

**Key insight from shaping this constraint:** Senior engineers cite the impressive number (50M). Staff engineers cite the operational number that actually constrained architecture (700K DAU), then use the larger number to show reach and stakes. Citing only the impressive number is a credibility risk — interviewers will probe for concurrency and expose the gap.

### Constraint 2 — Business / feature velocity

> 17LIVE shipped to production weekly, with a continuous roadmap of engagement features. This constraint had the most impact during seasonal events, when feature timing was directly tied to revenue windows. Any approach that paused or significantly slowed feature delivery was off the table — missing a seasonal launch window meant losing that revenue permanently. There was no way to recover it after the season ended.

**Key insight:** The reason something is a constraint rather than a preference is usually the non-recoverability of the consequence. Revenue from a missed seasonal launch can't be recovered — the season is over. Name non-recoverability explicitly when describing constraints.

### Constraint 3 — Team capacity

> We had two senior engineers and one junior engineer, running migration work in parallel with ongoing feature delivery. This constraint had the most impact because the team had to keep both tracks moving at the same time — there was no point when one could pause for the other. Any approach that required a dedicated rewrite team of five to ten engineers was off the table. Leadership had made a deliberate tradeoff — they chose to prioritize feature velocity over migration speed, which meant we couldn't pull that many engineers away from product teams.

**Key insight:** Frame resource limits as leadership tradeoffs, not as obstacles. "Leadership made a deliberate tradeoff — they chose to prioritize feature velocity over migration speed" sounds Staff-level. "We didn't have enough people" sounds Senior-level.

### Constraint 4 — Existing technical foundation

> The monolith was built on Go, MongoDB, etcd for configuration, GCP Pub/Sub for event messaging, and Redis for distributed caching. This constraint had the most impact in defining the scope of the migration — we were changing the architecture, not the stack. The infrastructure primitives were already production-proven. What was missing was the service decomposition layer and a clear service-to-service communication model. Any approach that required replacing the data layer, the config store, or the event backbone was off the table — those components were stable and shared across the entire monolith, so replacing them would have been a second migration on top of the first.

**Key insight:** Staff engineers name what they kept and what they changed. The fact that the infrastructure primitives already existed is why strangler fig was feasible with a three-person team. Constraint 3 and Constraint 4 connect — neither alone explains the architectural decision, but together they make it almost inevitable.

### Constraint 5 — Organizational / SRE dependency

> The SRE team owned networking, monitoring, and deployments. This constraint had the most impact because any approach that needed a new service-to-service protocol couldn't just be done by the platform team alone. SRE had their own roadmap and they controlled what new infrastructure could go into production. And service mesh wasn't something they already had. That was something I was proposing as part of the microservices architecture. So I had to build a working POC that showed two things: first, that gRPC was safe and ready for production, and second, that it gave SRE a clear path to adopt service mesh. That was the unlock. Without the POC, there was no way to move forward.

**Key insight:** Organizational dependencies are often disguised as technical ones. The real constraint wasn't "we had to use gRPC." It was "we had to earn SRE's trust with a POC to adopt gRPC." The POC also served as the vehicle for introducing service mesh to SRE, which they didn't have before. Naming the stakeholder's incentive structure (SRE has their own roadmap and gates adoption behind operational readiness) is Staff-level framing.

## Nine Verified Claims About the 17LIVE Work

During Layer 1, the narrative of the 17LIVE story was significantly reframed from the Session 1 diagnostic. The earlier framing was "supervisor asked me to lead the migration." The accurate framing is "I identified the architectural ownership vacuum and stepped into it proactively, because I believed architectural accountability was a Staff-engineer responsibility."

The nine verified claims that anchor the story:

1. Identified the monolith-as-bottleneck problem proactively — no one asked.
2. No engineer in the org had architectural ownership of the direction before I stepped in.
3. Proactively designed the solution before being asked.
4. Built the business case and escalated to supervisor and then VP.
5. Hexagonal architecture was my design.
6. gRPC adoption across the microservices ecosystem was a result of my POC and advocacy.
7. At the time, I believed taking on this architectural ownership was part of my Staff-engineer responsibility — not something extraordinary.
8. Over three years, led migration from zero to production microservices running in parallel with the legacy monolith — serving 50M registered users across multiple regions throughout the entire period, with no user-facing regressions during my tenure. Decomposed two legacy components that had high fan-out — each was used by many features, so decoupling produced broad architectural benefit. Established the rule that all new feature development would happen on the new architecture, which stopped the monolith from growing and created a migration flywheel. Migration was still actively decomposing legacy modules when I transitioned out.
9. My team was the first to adopt hexagonal architecture in production. I introduced the pattern across all feature teams as the standard design pattern for all new development and migration work. I established the rule that all new feature development org-wide would use the new architecture — so by the time I transitioned out, every new feature across engineering was being built on the hexagonal pattern, even in teams still maintaining legacy monolith code.

## The Ownership Framing — Staff vs. Senior mental model

The most important discovery of this session was the mental model behind my decision to take architectural ownership at 17LIVE.

**Senior mental model:** wait for the mandate, stay in scope.
> "This is above my pay grade." / "If leadership wanted this done, they would have hired an architect." / "I'll focus on my sprint work; if they want me to lead architecture, they'll ask."

**Staff mental model:** the title means I own the architectural problems in my domain, whether or not someone asked.
> "No one owns the architecture. I have the Staff title. Therefore this is my job."

I acted on the Staff mental model at 17LIVE without realizing it was a distinguishing behavior. This framing maps directly to competencies at every target company on my list (Uber, Databricks, JetBrains, ING, Adyen). Use it in behavioral questions like "tell me about a time you took ownership beyond your scope" or "describe a situation where you drove change without formal authority."

Opening phrase to internalize:
> "At 17LIVE, I observed that no engineer had taken architectural ownership of [specific problem]. I believed that was a responsibility that fell to me as a Staff engineer — so I proactively [what I did]."

## Overclaim Corrections — The Credibility Protection Muscle

Two overclaims were caught and corrected during this session before they made it into any interview material:

**Overclaim 1 (corrected):** "Zero user-facing downtime across a multi-year program" → "No user-facing regressions during my tenure." I don't have documented SLO measurements for "zero downtime," but "no user-facing regressions" is observable in error rates and user complaints and is defensible under interrogation.

**Overclaim 2 (corrected):** "Migration was completed" → "Migration was still actively decomposing legacy modules when I transitioned out; microservices and monolith were running in parallel." Strangler fig programs typically outlast individual engineers. Acknowledging the parallel state is more accurate and shows I understood the pattern correctly.

**Overclaim 3 (corrected):** "Components were blocking multiple feature teams" → "Components had high fan-out — each was used by many features, so decoupling produced broad architectural benefit." These are different claims entirely. The honest version is about architectural coupling, not team blocking.

**Lesson to carry forward:** Every reframing Claude offers, I challenge. Every claim in my interview narrative, I verify line by line. Interviewers at Adyen and Databricks are trained to probe for overclaiming, and the backpedal is the credibility hit — not the original claim. Better to get corrected ten times in coaching than once in front of a panel.

## Staff-level Filler Phrases Internalized This Session

1. "The constraint that mattered most here was..." — opener when introducing a named constraint
2. "What made this a hard constraint rather than a preference was that [consequence] couldn't be recovered after the fact." — the non-recoverability test
3. "Our [operational metric] was [number], on a platform reaching [larger metric]. The constraint that mattered for [decision] was [the operational one], because [why]." — how to cite scale honestly
4. "Leadership had made a deliberate tradeoff here — they chose to prioritize [X] over [Y], which meant that any architectural approach requiring [Z] was off the table." — how to frame resource constraints
5. "The technical foundation gave us [X, Y, Z] for free — those were already production-proven. What was missing was [the specific layer we had to add]. So our architectural scope was bounded to [the delta], not the full system." — how to scope technical constraints
6. "I prioritized the work with the highest [coupling fan-out / blast radius / cross-team impact], because doing that first produced broader benefit per unit of effort." — prioritization rationale
7. "At [company], I observed that no engineer had taken architectural ownership of [specific problem]. I believed that was a responsibility that fell to me as a Staff engineer — so I proactively [what I did]." — ownership framing opener

## Open Items for Session 3

- Layer 2: Alternatives (what approaches to the 17LIVE migration I considered and rejected — big-bang rewrite, parallel run with full feature parity, branch-by-abstraction, event interception, anti-corruption layer as standalone pattern)
- Layer 3: Tradeoffs (for each alternative, what it optimized for and what it sacrificed)
- Layer 4: Counterfactuals (what would have to change about the five constraints for me to pick a different alternative)
- Stitch all four layers into a 3–4 minute coherent delivery of the 17LIVE story
- Target companies in the plan doc need updating to Uber, Databricks, JetBrains, ING, Adyen (replace the earlier list including Preply, Nebius)

## Homework Before Session 3

1. Commit these notes to the repo.
2. Read through the five constraints out loud, twice. Not to memorize — to let the language patterns settle.
3. Draft a first attempt at Layer 2 Alternatives: list 3–4 approaches to the monolith→microservices migration that could have worked in principle. Name them explicitly (not just "the other way"). Examples of named alternatives: big-bang rewrite, parallel run with feature parity, branch-by-abstraction, event interception, anti-corruption layer as a standalone migration. Don't evaluate them yet — just name them and give a one-sentence description of what each would look like in practice at 17LIVE.
4. For each alternative, make a note of which of the five constraints would have ruled it out. This is pre-work for Layer 3 (Tradeoffs).

## Session Reflection — Coaching Notes

- Session ran 90 minutes, covered Layer 1 only. This is the correct pace for foundational work. Layers 2–4 will move faster once Layer 1 is in place.
- The story reframing (from "supervisor asked me" to "I identified the ownership gap") was the most valuable single outcome of the session. Changes the framing of every behavioral interview where the 17LIVE migration comes up.
- Challenging Claude's drafts for overclaims is a critical habit. Every draft Claude produces, I verify line by line.
- Transcription quality is a real issue for voice-based sessions. "SRE" was transcribed as "SI" and required mid-session correction. For future voice sessions: slow down on technical acronyms and proper nouns.

---
