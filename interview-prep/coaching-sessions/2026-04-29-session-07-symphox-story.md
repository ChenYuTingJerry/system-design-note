# Session 7 — Symphox Story Framework (Influence Without Authority)

**Date:** 2026-04-29
**Coach:** Claude
**Duration:** ~60 minutes
**Focus:** Build Symphox story framework using CARL structure, identify core problem, action, result, learning
**Status:** CARL framework locked. Full story delivered end-to-end once. Senior+ mode not yet built. Clean delivery scheduled for Session 8.

## Session Context

Week 2, Day 3. Started by reviewing resume v28 — two issues identified (header location, 17LIVE bullet 3 overclaim) but deferred to keep coaching momentum. Then built Symphox story framework from scratch using live Q&A to extract the core narrative.

## Resume v28 Review Notes (deferred)

Two issues identified but not yet fixed:
1. **Header:** "Relocating to the Netherlands" — Jerry confirmed Netherlands is top priority, so keep as is for now.
2. **17LIVE bullet 3:** "presenting a quantitative risk model to VP-level stakeholders" — still an overclaim. Needs fixing separately.

## Symphox Story — Core Facts

**The problem Jerry identified:**
- All services deployed manually across EC2, Lambda, and Step Functions
- No unified CI/CD process
- No IaC project to document provisioning scripts
- Each project had its own deployment workflow — silo problem
- Developers could modify source code directly on cloud — high risk, no version control
- Cloud architect team focused on infrastructure provisioning, couldn't see developer-side problems

**Who had authority but wasn't moving:**
Cloud architect team. They owned infrastructure but didn't identify developer workflow issues. Gap existed between infrastructure provisioning (their focus) and developer experience (Jerry's concern).

**How Jerry influenced without formal authority:**
1. Consulted with cloud architect tech leader to understand team responsibilities and boundaries
2. Got AWS account permissions for engineering teams
3. Built a POC to prove the solution works
4. Showed results to both cloud architect team and engineering teams
5. They adopted it

**Key insight:** POC-first approach. Same pattern as 17LIVE (built gRPC POC to unlock SRE support). Jerry builds proof first, then shows results. This is his influence-without-authority playbook.

## CARL Framework — Symphox Story

### Context (30 seconds)
At Symphox, I noticed there was no unified CI/CD process and no IaC. Each project had its own deployment workflow, which created silos. Developers were modifying code directly on cloud — no version control, high operational risk. The cloud architect team was focused on infrastructure provisioning and couldn't see these developer-side problems.

### Action (60 seconds)
I consulted with leaders in both the cloud architect team and engineering teams to understand the gap. After addressing the problem, I designed a unified CI/CD workflow and used Terraform to build an IaC solution. I also introduced Kubernetes to unify the deployment environment, so all services — whether on EC2, Lambda, or Step Functions — could run on a consistent platform. I built a POC first to prove it worked, then introduced it to both teams. After proving the concept, I trained the engineering teams to adopt the new unified workflow.

### Result (30 seconds)
Services could be deployed safely and repeatedly. Each project had version control. CI/CD became self-service. We reduced communication time with the cloud architect team because teams could provision infrastructure independently. I also transferred knowledge to two engineers to maintain the IaC project together, so they could support each other and protect quality through code review.

### Learning (15 seconds)
What I learned is that maintainability and process clarity matter more than fancy technology. Transparency and a support culture — where engineers help each other — make teams go faster than any individual tool.

## Additional Detail (to add to story in Session 8)

Jerry mentioned one detail that was missing from the first delivery:

**Kubernetes introduction:** After designing unified CI/CD + Terraform IaC, Jerry also introduced Kubernetes to unify the deployment environment. This was the key architectural decision that allowed all services (previously scattered across EC2, Lambda, Step Functions) to run on a consistent platform with the unified CI/CD process and deployment workflow.

This detail must be included in the clean delivery tomorrow.

## Story Structure (CARL — 2 to 2.5 minutes)

**Opening:** "At Symphox, I noticed there was no unified CI/CD process and no IaC..."
**Context:** Silo problem, manual deployments, cloud architect team couldn't see developer problems
**Action:** Consulted leaders, designed unified CI/CD + Terraform IaC + Kubernetes, built POC, trained teams
**Result:** Safe deployments, self-service CI/CD, reduced communication, knowledge transfer
**Learning:** Process + maintainability over fancy technology, transparency + support culture = speed

## Staff vs Senior+ Delivery Modes (to build in Session 8)

**Staff mode (influence without authority):**
- Lead with: "I identified a gap that nobody owned. I believed it was my responsibility to fix it."
- Emphasis: How you influenced without authority (POC-first approach, consulting leaders, building consensus)
- For: "Tell me about a time you drove change without formal authority"

**Senior+ mode (technical execution):**
- Lead with: "At Symphox, I designed and built a unified CI/CD platform on Kubernetes with Terraform IaC."
- Emphasis: What you built, how it worked, technical decisions
- For: "Tell me about a platform you built" or "How would you approach unified CI/CD for multiple teams?"

## Influence Without Authority Pattern (confirmed)

Jerry's playbook for influence without authority:
1. Consult with the relevant team leader to understand boundaries and responsibilities
2. Identify what falls within your scope vs. what needs their support
3. Build a POC independently using available resources
4. Show results to both teams
5. Let the results drive adoption

This pattern appeared in both 17LIVE (gRPC POC → SRE adopted service mesh) and Symphox (CI/CD POC → cloud architect team aligned). It's a repeatable Staff-level behavior worth naming explicitly in interviews.

## Open Items for Session 8

1. Clean cold delivery of Symphox story (include Kubernetes detail)
2. Build Staff mode delivery (influence without authority framing)
3. Build Senior+ mode delivery (technical execution framing)
4. Start billing story framework (team leadership + dysfunction management)

## Schedule Update

**Week 2 compression:**
- Session 7 (today): Symphox framework locked ✅
- Session 8 (Apr 30): Symphox clean delivery + both modes + start billing story
- Session 9 (May 1): Billing story + common questions + final polish
- By May 1: Ready for Week 3 (system design + real mock)

## Writing Rules (Confirmed)

- No em dashes
- B2 conversational English
- American English throughout
- No overclaims
- Accuracy is non-negotiable
