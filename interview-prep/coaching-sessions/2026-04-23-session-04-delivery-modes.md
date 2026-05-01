# Session 4 — 4-Layer Story Delivery + Senior+ Mode Introduction (17LIVE)

**Date:** 2026-04-23
**Coach:** Claude
**Duration:** ~90 minutes
**Focus:** Cold constraint delivery (validation), full 4-layer story compressed delivery, Senior+ delivery mode introduction
**Status:** 4-layer story locked. Compressed 3-minute version delivered clean. Senior+ mode prep begins Session 5.

## Session Context

Fresh start after Session 3. Constraints were practiced cold in the morning (better energy). Full 4-layer story delivered end-to-end without stopping. No major looping. Significant improvement in verbal fluency from Session 3 to Session 4.

## Cold Constraint Delivery (5 constraints, ~3.5 minutes)

All five constraints delivered from memory, one after another. Transcription noise present (system artifacts, not speech issues).

### Constraint 1 (User scale and operational load)
Delivered clean. Numbers correct (700K DAU, 2M MAU, 50M registered, multiple regions). Peak Asia-evening windows named. Availability risk and brand incident consequence clear.

### Constraint 2 (Feature velocity)
Delivered clean. Weekly production releases, seasonal revenue windows, non-recoverability of missed revenue windows articulated clearly.

### Constraint 3 (Team capacity)
Looped slightly on "any decision, any approach" but caught it and moved on. Core idea solid: 3 engineers, parallel tracks, 5-10 engineer rewrite team ruled out.

### Constraint 4 (Technical foundation)
Delivered with transcription noise (Golem → Go, EDCD → etcd, pops up → Pub/Sub, text stack → tech stack). Ideas clear: production-proven stack, scope bounded to architecture not stack replacement.

### Constraint 5 (SRE dependency)
Delivered clean. SRE ownership of networking/monitoring/deployments, service mesh as proposal (not pre-existing), POC as unlock mechanism for both gRPC and service mesh adoption.

## Full 4-Layer Story — Compressed Delivery (3-4 minutes)

Delivered end-to-end without stopping.

**Layer 1 (Opening + Constraints):** Ownership gap framing clear. All five constraints named with numbers.

**Layer 2 + 3 (Alternatives + Tradeoffs):** Three alternatives named (big-bang rewrite, parallel run, branch-by-abstraction). Tradeoffs articulated: big-bang sacrifices risk mitigation, strangler fig optimizes for incremental validation and continuous value delivery.

**Layer 4 (Counterfactual):** "If I had 5-10 dedicated engineers, big-bang rewrite becomes viable. But with all five constraints in place, strangler fig was the right call."

**Result:** Over three years, went from zero to production microservices. Designed hexagonal architecture, got gRPC adopted through POC, established rule that all new development uses new architecture. No user-facing regressions during tenure.

## Verbal Delivery Progress

**Session 3 observations:** Looping on repeated phrases, fatigue-driven repetition, pushing through broken sentences.

**Session 4 observations:** Minimal looping, fluent delivery, ideas flow naturally, framework internalized (not reciting).

**Key difference:** Fresh energy (morning practice) + ownership of ideas (not memorized words) = smoother delivery.

**Transcription artifacts present:**
- "Solar per active" → "so I proactively"
- "RADIS" → "Redis"
- "Pops up" → "Pub/Sub"
- "Runway" → "run"
- "Tenu" → "tenure"

These are voice system transcription errors, not speech errors. Actual delivery was clean.

## Fluency Drills Effectiveness

**What worked today:**
1. **Practice when fresh** — morning session produced noticeably better delivery than yesterday's afternoon session
2. **Own the ideas, not words** — when you explained constraints in your own phrasing, no stumbling; when you tried exact wording, slight hesitation
3. **Stop-and-restart technique** — you caught yourself on Constraint 3 repetition and moved on cleanly instead of looping

**What still needs work:**
- Slight rush at the very end of the story (compressed "deployment" instead of "development")
- One instance of "runway" instead of "run" — but caught immediately

## Senior+ Delivery Mode — Introduction

Original plan: Staff-only delivery (emphasis on ownership gap, cross-org influence, decision framework).

New requirement: Two delivery modes for every story to match application ratio (2 Staff + 5 Senior+/Lead + 2 Senior per 10 applications).

**Staff mode (what you delivered today):**
- Lead with: "No engineer had taken architectural ownership. I believed that fell to me as Staff."
- Emphasis: Cross-org influence, ownership proactivity, decision-making framework
- For: Staff-level interviews, questions about "taking ownership beyond scope"

**Senior+ mode (to practice next session):**
- Lead with: "I designed and led a monolith-to-microservices migration using strangler fig."
- Emphasis: Technical execution, hands-on architecture design, team coordination, incremental delivery
- For: Senior/Senior+ interviews, questions about "designing systems at scale"

**Same facts, different emphasis.** Not a different story. The four layers remain the same. Only the opening framing and narrative weight changes.

Example contrast:
- Staff: "I identified the ownership gap and stepped into it as a Staff engineer"
- Senior+: "I designed the strangler fig approach with hexagonal architecture and gRPC communication layer"

## Open Items for Session 5

1. One more cold delivery of 4-layer story (validate overnight retention)
2. Build the **Senior+ delivery mode** for 17LIVE story — same facts, emphasis on technical execution
3. Start the **60-second self-introduction** with ownership framing
4. If time: introduce Symphox story (influence without authority)

## Homework Before Session 5

1. Read the 4-layer story (compressed version) aloud once in the morning when fresh
2. Think about: "If someone asked me 'how did you actually design the hexagonal architecture?', what would I say?" — this is the Senior+ deep-dive
3. Mentally practice the stop-and-restart technique: if you catch yourself repeating, pause and restart
4. Do not memorize exact words. Own the ideas.

## Session Reflection — Coaching Notes

- Session ran 90 minutes, focused narrowly on constraint validation and full story delivery. Correct scope.
- Dramatic improvement in fluency from Session 3 to Session 4. Morning practice + fresh energy makes a measurable difference. This should become standard: practice constraints in the morning.
- The compressed 3-minute version is the right length for behavioral interviews. You have both the detailed version (for follow-up probes) and the compressed version (for core answer). Build this dual-version pattern into Symphox and billing stories.
- Looping is nearly gone. Stop-and-restart technique is working. Transcription noise is just system artifacts, not speech patterns.
- Ownership-gap framing is solid and internalized. Ready to layer in Senior+ execution-focused framing without losing the Staff framing.
- You own the 4-layer story. Next phase is building flexibility: same story, different emphasis for different interview contexts.
