# Session 6 — Senior+ Delivery Mode (17LIVE) + Mode Comparison

**Date:** 2026-04-28
**Coach:** Claude
**Duration:** ~90 minutes
**Focus:** Build Senior+ delivery mode for 17LIVE story, validate both modes (Staff vs Senior+), clarify when to use each mode
**Status:** Both Staff and Senior+ delivery modes locked. Ready for Symphox story framework in Session 7.

## Session Context

Week 2, Day 2. Confirmed all three Session 5 Claude Code prompts were committed (session notes, ChatGPT prompt, Claude coach prompt). Started with cold delivery of compressed 4-layer story to validate overnight retention. Then built Senior+ delivery mode through multiple iterations, including ChatGPT review and comparison. Final versions locked after iterative refinement.

## Staff vs Senior+ Delivery Mode — Core Difference

**Same story. Different door in.**

- **Staff mode:** Opens with ownership gap. Shows initiative, decision framework, Staff-level mindset.
- **Senior+ mode:** Opens with technical execution. Shows architecture depth, hands-on delivery.

Both use the same facts. Only the opening framing and emphasis changes.

**Key rule:** Don't mix modes in the same answer. Pick one based on the question, deliver it clean.

## When to Use Each Mode

**Use Staff mode for:**
- "Tell me about a time you took ownership beyond your scope"
- "Influence without authority" questions
- Questions about identifying problems proactively
- Staff-level interviews

**Use Senior+ mode for:**
- "Tell me about a system you designed"
- "How would you approach decomposing a monolith?"
- Architecture interviews
- Senior/Senior+ level interviews

## Staff Mode — Full Delivery (Locked)

At 17LIVE, I noticed the monolith was becoming a bottleneck, and there wasn't really a clear owner driving the decomposition. I believed that as a Staff engineer, that was my responsibility to fix. So I stepped in, scoped the problem, and proposed a solution to my manager and the VP.

There were a few key constraints.

First, we had around 50K to 150K DAU on a globally distributed platform. Traffic was very spiky, especially during Asia evening hours. So downtime was basically not acceptable.

Second, we were shipping features every week, especially during seasonal events, so we couldn't pause product development.

Third, the team was small — just three engineers — so we had very limited capacity.

Also, the tech stack was already stable in production, and infrastructure decisions were owned by SRE, so we had to align with them.

Given all that, I looked at a few options.

Big-bang rewrite was too risky and would have needed a bigger team.

Parallel run… we wouldn't really get value until everything was built, so it wasn't a great fit.

Branch-by-abstraction also didn't work well, because we'd basically be maintaining two versions of the same logic.

If we had five to ten engineers, big-bang rewrite might have been viable. But given our constraints… strangler fig made the most sense.

So we extracted services gradually, ran them alongside the monolith, validated them in production, and then removed the old code step by step.

It's slower, but it gives strong risk isolation, and we could start getting value early.

We also made it a rule that all new features had to go into the new services.

So over time, the monolith stopped growing, and the system naturally shifted toward microservices.

On the architecture side, I introduced hexagonal architecture to separate business logic from infrastructure.

And I built a small gRPC PoC, which helped the SRE team move toward a service mesh.

After about three years, most of the system was running on microservices, and we didn't see any user-facing regressions.

Looking back… I think the key was really aligning the solution with the constraints.

**Runtime:** ~3 minutes at comfortable speaking pace

## Senior+ Mode — Full Delivery (Locked)

So at 17LIVE, I led a migration from a monolith to microservices. It took around three years. The system had around 50K to 150K DAU across regional deployments in Asia, US, and international markets. Traffic was pretty spiky, especially during Asia evening hours.

The main challenge was… we couldn't break production, but at the same time, we still had to keep shipping features.

So instead of doing a big rewrite — which is pretty risky — we went with the Strangler Fig approach. Basically, we extracted services one by one. We put them behind an API gateway, and let them run alongside the monolith. And we only removed the old code after we were confident it was stable in production.

That helped us keep the risk low. And it also meant we could adjust the design as we learned from real traffic.

On the architecture side, I introduced Hexagonal Architecture, mainly to separate business logic from infrastructure. That made the services much easier to maintain and evolve.

I also built a small gRPC PoC, which later helped the SRE team move toward a service mesh.

Another thing we did was… we made it a rule that all new features had to go into the new services. So over time, the system naturally moved away from the monolith, without needing a separate migration push.

If we had a bigger team — say five to ten engineers — a big-bang rewrite might have been viable. But given our constraints… strangler fig made the most sense.

And yeah… after about three years, most of the system was running on microservices, with no user-facing regressions during the migration.

**Runtime:** ~2 minutes at comfortable speaking pace

## Iteration Notes — How Versions Were Refined

Multiple iterations done this session, including ChatGPT review. Key lessons:

1. **ChatGPT made it too formal.** Words like "large-scale", "thorough validation", "greatly improving" sound written, not spoken. Rejected.
2. **Natural hesitations are good.** "pretty spiky", "basically", "And yeah" make the answer sound conversational and confident, not scripted.
3. **Specificity builds credibility.** "multiple regions" is better than "more than 100 countries."
4. **Overclaim corrections applied:**
   - "zero user-facing regressions" → "no user-facing regressions"
   - "completed the entire migration" → "most of the system was running on microservices"
   - "production-ready" → "production"
   - "large-scale" → removed
5. **Ownership-gap framing is Staff-specific.** One sentence difference in opening separates Staff from Senior+ mode.
6. **Counterfactual belongs in both modes.** "If we had five to ten engineers, big-bang rewrite might have been viable" gives the answer Staff-level depth.

## Open Items for Session 7

1. Symphox story framework — influence without authority (CARL structure)
2. Answer three framing questions before building the story:
   - What was the core problem at Symphox?
   - Who had formal authority but wasn't moving?
   - How did you influence without that authority?
3. Build Symphox story with Staff + Senior+ modes

## Writing Rules (Confirmed)

- No em dashes in written materials (natural speech pauses are fine)
- B2 conversational English
- 50K-150K DAU (not 700K)
- multiple regions (not "more than 100 countries")
- 15+ years software engineering, 6+ years platform engineering
- American English throughout
- No overclaims: "no user-facing regressions" not "zero", "most of the system" not "completed the migration"

## Schedule Update

**Week 2 compression on track:**
- Session 6 (today): 17LIVE both modes locked ✅
- Session 7 (tomorrow): Symphox story + framework
- Session 8-9 (Apr 30-May 1): Billing story + common questions + all three stories polished
- By May 1: Ready for Week 3 (system design + real mock)
