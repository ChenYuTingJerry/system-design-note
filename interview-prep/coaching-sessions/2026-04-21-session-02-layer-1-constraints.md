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

Order matters: can't articulate tradeoffs without alternatives, can't articulate counterfactuals without constraints. Build in order.

## Layer 1 — Five Constraints (17LIVE monolith→microservices migration)

### Constraint 1 — User scale and operational load

> 700K DAU and 2M MAU on a platform reaching 50M registered users across 133 countries. The constraint bit hardest during peak Asia-evening windows. Any architectural decision that risked whole-system availability during peak was off the table, because an outage would trigger a global brand incident across 133 countries.

**Key insight from shaping this constraint:** Senior engineers cite the impressive number (50M). Staff engineers cite the *operational* number that actually constrained architecture (700K DAU), then contextualize the larger numbers as reach/stakes. Citing only the impressive number is a credibility risk because interviewers will probe for concurrency and expose the gap.

### Constraint 2 — Business / feature velocity

> 17LIVE shipped to production weekly, with a continuous roadmap of engagement features. The constraint bit hardest during seasonal events, when feature timing was tied to revenue windows. Any architectural approach that paused or significantly slowed feature delivery was off the table, because missing the launch window for a seasonal feature meant losing the associated revenue — that revenue couldn't be recovered later.

**Key insight:** The reason something is a *constraint* rather than a *preference* is usually the non-recoverability of the consequence. Revenue from a missed seasonal launch can't be recovered — the season is over. Name non-recoverability explicitly when describing constraints.

### Constraint 3 — Team capacity

> Two senior engineers and one junior engineer, running migration work in parallel with ongoing feature delivery. The constraint bit hardest because the team had to sustain both the migration and the feature roadmap simultaneously — there was no moment where one could pause for the other. Any architectural approach that required a dedicated rewrite team of 5–10 engineers was off the table, because leadership had approved the migration in principle but was unwilling to slow feature velocity enough to free up that many engineers from product teams.

**Key insight:** Frame resource limits as leadership tradeoffs, not as obstacles. "Leadership made a deliberate tradeoff — they chose to prioritize feature velocity over migration speed" sounds Staff-level. "We didn't have enough people" sounds Senior-level.

### Constraint 4 — Existing technical foundation

> The monolith was built on Go, MongoDB, etcd for configuration, GCP Pub/Sub for event messaging, and Redis for distributed caching. The constraint bit hardest in that we were migrating architecture, not stack — the infrastructure primitives were proven in production, but the service decomposition and service-to-service communication layer was missing. Any architectural approach that required replacing the data layer, the config store, or the event backbone was off the table, because those components were operationally stable and shared across every part of the monolith — replacing them would have been a second migration layered on top of the first.

**Key insight:** Staff engineers name what they kept and what they changed. The fact that the infrastructure primitives already existed is why strangler fig was feasible with a 3-person team. Constraint 3 and Constraint 4 connect — neither alone explains the architectural decision, but together they make it almost inevitable.

### Constraint 5 — Organizational / SRE dependency

> The SRE team owned the L7 infrastructure layer — service mesh, load balancing, and production networking. Any architectural approach that required a new service-to-service protocol could not be executed unilaterally by the platform team, because SRE had their own roadmap and gated new infrastructure adoption behind operational readiness. The unlock mechanism was delivering a working POC that demonstrated safety and operational readiness before SRE would commit to supporting gRPC in production.

**Key insight:** Organizational dependencies disguised as technical ones are common. The real constraint wasn't "we had to use gRPC" — it was "we had to earn SRE's trust with a POC to adopt gRPC." Naming the stakeholder's incentive structure (SRE has own roadmap, gates adoption behind operational readiness) is Staff-level framing.

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
8. Over three years, led migration from zero to production microservices running in parallel with the legacy monolith — serving 50M registered users across 133 countries throughout, with no user-facing regressions during my tenure. Decomposed two legacy components that had high fan-out — each used by many features, so decoupling produced broad architectural benefit. Established the rule that all new feature development would happen on the new architecture, which stopped the monolith from growing and created a migration flywheel. Migration was still actively decomposing legacy modules when I transitioned out.
9. My team was the first to adopt hexagonal architecture in production. I introduced the pattern across all feature teams as the standard they would follow when migrating. I established the rule that all new feature development org-wide would use the new architecture — so by the time I transitioned out, every new feature across engineering was being built on the hexagonal pattern, even in teams still maintaining legacy monolith code.

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

**Overclaim 1 (corrected):** "Zero user-facing downtime across a multi-year program" → "No user-facing regressions during my tenure." I don't have documented SLO measurements for "zero downtime," but "no user-facing regressions" is observable in error rates and user complaints and defensible under interrogation.

**Overclaim 2 (corrected):** "Migration was completed" → "Migration was still actively decomposing legacy modules when I transitioned out; microservices and monolith were running in parallel." Strangler fig programs typically outlast individual engineers. Acknowledging the parallel state is more accurate and shows I understood the pattern correctly.

**Overclaim 3 (corrected):** "Components were blocking multiple feature teams" → "Components had high fan-out — each was used by many features, so decoupling produced broad architectural benefit." Different claim entirely. The honest version is about architectural coupling, not team blocking.

**Lesson to carry forward:** Every reframing Claude offers me, I challenge. Every claim in my interview narrative, I verify. Interviewers at Adyen and Databricks are trained to probe for overclaiming, and the backpedal is the credibility hit — not the original claim. Better to get corrected ten times in coaching than once in front of a panel.

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
- Correcting Claude's drafts on two overclaims is a critical habit. Every draft I give myself, I verify line by line.
- Transcription quality is a real issue for voice-based sessions. "SRE" was transcribed as "SI" and required mid-session correction. For future voice sessions: slow down on technical acronyms and proper nouns.
