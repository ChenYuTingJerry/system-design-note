# Session 6 — Senior+ Delivery Mode (17LIVE) + Mode Comparison

**Date:** 2026-04-28
**Coach:** Claude
**Duration:** ~60 minutes
**Focus:** Build Senior+ delivery mode for 17LIVE story, validate both modes (Staff vs Senior+), clarify when to use each mode
**Status:** Both Staff and Senior+ delivery modes locked. Ready for Symphox story framework in Session 7.

## Session Context

Week 2, Day 2. Confirmed all three Session 5 Claude Code prompts were committed (session notes, ChatGPT prompt, Claude coach prompt). Started with cold delivery of compressed 4-layer story to validate overnight retention. Then built Senior+ delivery mode.

## Cold Delivery of Compressed 4-Layer Story (Staff Mode)

Delivered end-to-end. All six sections present:
- Opening (ownership gap)
- Constraints (5 including SRE dependency + seasonal revenue)
- Alternatives (3 approaches)
- Your choice + Tradeoffs (strangler fig)
- Counterfactual (if 5-10 engineers)
- Result (hexagonal architecture, gRPC, no regressions)

**Observations:** Ideas flow naturally. Framework internalized. Minor transcription artifacts ("dry PC" instead of "gRPC", "constraints in place" instead of "all five constraints in place") but delivery is clean. No major looping. Ready to move to alternate delivery mode.

## Staff vs Senior+ Delivery Mode Comparison

### Staff Mode (Ownership-Gap Framing)

**Opening:**
"At 17LIVE, I noticed that nobody had really taken ownership of decomposing the monolith into microservices. The architecture was becoming a bottleneck. I believed that as a Staff engineer, it was my responsibility to step in and figure this out. So I did."

**Flow:** Opening → Constraints (5) → Alternatives (3) → Your choice + Tradeoffs → Counterfactual → Result

**Emphasis:** Why you stepped in, decision-making framework, constraints that shaped the choice

**Best for:** Behavioral questions like "Tell me about a time you took ownership beyond your scope" or "Describe a situation where you drove change without formal authority"

**Key insight:** Shows Staff-level ownership mentality — you identified the gap and proactively stepped in

### Senior+ Mode (Technical-Execution Framing)

**Opening:**
"At 17LIVE, I designed and led a monolith-to-microservices migration using the strangler fig pattern over three years."

**Flow:** Opening → Challenge → Approach → Key technical decisions → Results

**Emphasis:** What you built, how you designed it, why the technical choices worked

**Best for:** System design questions like "Tell me about a system you designed" or "Describe an architecture you've built"

**Key insight:** Shows Senior+-level technical depth — you made specific architectural decisions and executed them

### Core Difference

**Same story.** Different door in.

- **Staff mode:** "Nobody owned this problem, so I stepped in" (shows initiative + decision framework)
- **Senior+ mode:** "I designed this migration using strangler fig" (shows technical execution + architecture depth)

Both use the same facts. The 50K-150K DAU, three engineers, constraints, strangler fig choice — all the same. Only the opening framing changes.

## Senior+ Mode Delivery (Locked)

Clean delivery of complete Senior+ version:

"At 17LIVE, I designed and led a monolith-to-microservices migration using the strangler fig pattern over three years. We had 50K to 150K DAU across 133 countries, with spikes during peak Asia-evening hours. The challenge was moving to microservices without disrupting live traffic or pausing feature releases. The approach: Extract services one at a time using an API gateway router. Each new service ran in parallel with the monolith. We validated the output matched the legacy system exactly before abandoning old code. This gave us incremental value and the ability to course-correct. I designed the hexagonal architecture pattern for service boundaries. I built a gRPC POC that convinced SRE to adopt service mesh. I established the rule that all new development had to use the new architecture, creating a migration flywheel. Over three years, we went from zero to production microservices. Every new feature was being built on the modern stack. No user-facing regressions during my tenure."

**Runtime:** ~2.5 minutes (slightly faster than Staff mode due to skipping constraints section)

**Quality:** Clean, natural, no looping, all key facts present

## When to Use Each Mode

**Use Staff mode (ownership gap) for:**
- Behavioral interviews: "Tell me about ownership"
- "Influence without authority" questions
- Questions about identifying problems proactively
- Discussions about your Staff-level mindset

**Use Senior+ mode (technical execution) for:**
- System design interviews: "Design a monolith-to-microservices migration"
- Architecture interviews
- Questions about what you built
- Questions about technical decision-making
- When the interviewer asks: "Tell me about a system you designed"

**Key rule:** Don't mix modes in the same answer. Pick one based on the question, then deliver it clean.

## Interview Application Examples

**Question:** "Tell me about a time you took ownership beyond your scope."
**Answer:** Use Staff mode (ownership gap opening)

**Question:** "How would you approach decomposing a monolith?"
**Answer:** Use Senior+ mode (technical execution opening)

**Question:** "Describe your most complex project."
**Answer:** Could use either, but Senior+ is safer (focuses on what you built, not why you stepped in)

## Open Items for Session 7

1. **Symphox story framework** — influence without authority (CARL structure)
2. Answer three framing questions:
   - What was the core problem at Symphox?
   - Who blocked you (had formal authority but wasn't moving)?
   - How did you win without that authority?
3. Build Symphox story with Staff + Senior+ modes

## Key Learnings from Session 6

1. **Same story, different framing = two interview modes.** Both Staff and Senior+ modes use identical facts. Only the opening changes. This is the model for Symphox and billing stories.

2. **Ownership framing is Staff-specific.** "Nobody owned this, so I stepped in" is powerful for Staff interviews but sounds defensive in Senior+ interviews where you're expected to have designed systems anyway.

3. **Delivery is solid when you own ideas, not words.** Both Staff and Senior+ modes flowed naturally because you explained ideas in your own words, not reciting memorized sentences.

## Writing Rules (Confirmed)

- No em dashes
- B2 conversational English
- 50K-150K DAU (not 700K)
- 133 countries across regions (not 7 regions — 133 is the country count)
- 15+ years software engineering, 6+ years platform engineering
- American English throughout

## Schedule Update

**Week 2 compression on track:**
- Session 6 (today): 17LIVE both modes locked ✅
- Session 7 (tomorrow): Symphox story + framework
- Session 8-9 (Apr 30-May 1): Billing story + common questions + all three stories polished
- By May 1: Ready for Week 3 (system design + real mock)
