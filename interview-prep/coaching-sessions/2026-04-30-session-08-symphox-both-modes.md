# Session 8 — Symphox Story (Both Modes Locked)

**Date:** 2026-04-30
**Coach:** Claude
**Duration:** ~90 minutes
**Focus:** Deliver Symphox story in both Staff and Senior+ modes, confirm CARL structure works for both
**Status:** Symphox Staff mode locked. Symphox Senior+ mode locked. Ready for Billing story in Session 9.

## Session Context

Week 2, Day 4. Started with cold delivery of Symphox story (Staff mode). Successfully delivered with all CARL elements. Then built Senior+ mode, discovered need to include Context section in both modes. Refined Senior+ mode to include full CARL structure. Both modes now locked.

## Key Discovery This Session

**Staff and Senior+ modes both use full CARL structure** (Context, Action, Result, Learning). The difference is NOT the structure — it's the opening framing:

- **Staff mode opening:** Ownership framing ("I believed that as a platform engineer, that was my responsibility to address")
- **Senior+ mode opening:** Technical execution framing ("I designed and built a unified CI/CD platform...")

After the opening, both modes follow identical CARL structure.

## Symphox Story — Staff Mode (Locked)

**Opening (Ownership framing):**
At Symphox, I noticed there was no unified deployment process and no infrastructure as code. Each project had its own development workflow, which created silos. I believed that as a platform engineer, that was my responsibility to address.

**Context:**
So I consulted with leaders in both the cloud architect team and the engineering teams to understand the gap.

**Action:**
After addressing the problem, I designed a unified development environment using Kubernetes and unified the CI/CD workflow using Terraform as an IaC solution. So all the services — whether on EC2, Lambda, or Step Functions — could run on a consistent platform. I built a POC first to prove it worked, then introduced it to both teams. After proving the concept, I built the whole architecture, and then I trained the engineering teams to adopt the new unified workflow.

**Result:**
Services could be deployed safely and repeatedly. Each project had version control. CI/CD became self-service. We reduced communication time with the cloud architect team because teams could provision infrastructure on their own. I also transferred knowledge to two engineers to maintain the IaC project together, so they could support each other and protect quality through code review.

**Learning:**
What I learned is that maintainability and process clarity matter more than fancy or complex technology. Transparency and a support culture — where engineers help each other — eliminate silo problems and make teams go faster.

**Runtime:** ~2.5 minutes

## Symphox Story — Senior+ Mode (Locked)

**Opening (Technical execution framing):**
At Symphox, services were deployed manually across EC2, Lambda, and Step Functions. Each project had its own CI/CD workflow, creating silos. Basically, no infrastructure as code.

**Action:**
So I designed and built a unified CI/CD process using Terraform to build an IaC solution and a unified Kubernetes environment using AWS EKS. So all services — whether on EC2, Lambda, or Step Functions — could deploy to a consistent platform. Services could be deployed safely and repeatedly. Each project had version control. CI/CD became self-service.

We reduced communication time with the cloud architect team because teams could provision infrastructure independently. I also transferred knowledge to two engineers to maintain the IaC project together, so they could support each other and protect code quality.

**Result / Learning:**
What I learned is that maintainability and process clarity — and unifying the tech stack — are most important for building an IDP. And transparency and a support culture where engineers help each other make engineering teams go faster.

**Runtime:** ~1.5 minutes

## Key Differences Between Staff and Senior+ Modes

| Element | Staff Mode | Senior+ Mode |
|---------|-----------|-------------|
| **Opening framing** | "I believed...that was my responsibility" (ownership) | "I designed and built..." (execution) |
| **Emphasis** | How you influenced without authority (POC-first, consulting leaders) | What you built and why it works (Kubernetes, Terraform, unified platform) |
| **For interview questions** | "Tell me about driving change without formal authority" | "Tell me about a platform you built" or "How would you design a unified CI/CD system?" |
| **Interview level** | Staff-level | Senior+/Senior level |
| **CARL structure** | Full (Context, Action, Result, Learning) | Full (Context, Action, Result, Learning) |
| **Key details highlighted** | Influence strategy, knowledge transfer, support culture | Kubernetes choice, Terraform IaC, AWS EKS, self-service CI/CD |

## Structure Insight (Important)

**Both modes use the same CARL framework.** The difference is not structural — it's the opening framing and where emphasis falls:

- **Staff mode:** Lead with ownership gap, then explain how you solved it
- **Senior+ mode:** Lead with the solution, then explain why it works

This pattern works for all three stories (17LIVE, Symphox, Billing).

## Current State of All Three Stories

### 1. 17LIVE (Complete)
- ✅ Staff mode locked (3 minutes, ownership-gap framing, six sections)
- ✅ Senior+ mode locked (2 minutes, technical-execution framing, condensed)
- ✅ Four layers (Constraints, Alternatives, Tradeoffs, Counterfactuals)
- ✅ No overclaims

### 2. Symphox (Complete)
- ✅ Staff mode locked (2.5 minutes, ownership framing, CARL structure)
- ✅ Senior+ mode locked (1.5 minutes, technical execution, CARL structure)
- ✅ Kubernetes detail included
- ✅ No overclaims

### 3. Billing (Not started)
- ❌ Neither mode
- ❌ Framework not built
- ❌ Facts not organized

## Session 9 Plan (Tomorrow — May 1)

1. **Billing story framework** — answer three questions:
   - What was the core problem? (team dysfunction, ownership crisis)
   - What was your formal authority? (EM role, you had it)
   - What did you do? (restructure, establish processes, align team)

2. **Billing story delivery** — CARL structure, both Staff and Senior+ modes

3. **Week 2 wrap** — final polish on all three stories

4. **Common questions** (if time):
   - "Why did you leave your last role?"
   - "Tell me about your EM to IC transition"
   - "Why are you relocating to Europe?"

## Writing Rules (Confirmed)

- No em dashes
- B2 conversational English
- American English throughout
- No overclaims
- Accuracy is non-negotiable

## Resume Status (Still Flagged)

**Two items not yet fixed:**
1. 17LIVE bullet 3: "presenting a quantitative risk model" — overclaim
2. Summary: "intentionally returning to an Individual Contributor role" — preferred: "focused on full-time IC platform engineering work"

**New findings from document review:**
- OpenTelemetry bullet is accurate ✅
- Jaeger mentioned in session but not in resume — confirm if accurate
- Self-healing probes + KEDA in LinkedIn but not resume — confirm if accurate
- Python microservices bullet in LinkedIn but not resume — confirm if should be added

## Schedule Status

**Week 2 (compression mode — on track):**
- Session 6 (Apr 28): 17LIVE both modes locked ✅
- Session 7 (Apr 29): Symphox framework locked ✅
- Session 8 (Apr 30): Symphox both modes locked ✅
- Session 9 (May 1): Billing story + Week 2 wrap (TARGET)

**By May 1:** All three stories locked, ready for Week 3 (system design + real mock interview)
