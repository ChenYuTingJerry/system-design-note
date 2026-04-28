# System Design Practice — Week 2, Session 1

## Topic: Internal Developer Platform (IDP)

**Date:** 2026-04-28
**Score:** 6.5 / 10
**Phase:** Introductory (Week 2)

---

## What Is an Internal Developer Platform (IDP)?

An Internal Developer Platform is a self-service layer built by a platform engineering team that abstracts away infrastructure complexity and standardizes how developers build, deploy, and operate software. It sits between developers and the underlying infrastructure (Kubernetes, cloud services, CI/CD tooling), providing a unified interface so developers can ship code without needing deep infrastructure expertise.

**Why companies build IDPs:**

- Reduce deployment friction (slow builds, manual processes, SRE bottlenecks)
- Standardize workflows across teams using different tech stacks
- Enable self-service infrastructure provisioning (databases, queues, secrets)
- Improve reliability (fewer failed deployments, faster rollbacks)
- Free SRE/platform teams from repetitive toil

**What a successful IDP looks like:**

| Metric | Before IDP | After IDP |
|--------|-----------|-----------|
| Deployment time | 2 hours | 15 minutes |
| Rollback rate | 15% | < 5% |
| SRE toil (manual provisioning) | 40% of time | 10% of time |
| Developer self-service rate | 0% | 80% of teams |

---

## Scenario

> Your company has 200 engineers across 15 teams. Deployments are slow (average 2 hours from merge to production), error-prone (15% rollback rate), and every team does it differently. Engineering leadership asks you to design an Internal Developer Platform that standardizes deployments, provides self-service infrastructure, and improves developer productivity.

### Clarified Constraints

- **Infrastructure:** Kubernetes (EKS/GKE/AKS), multiple clusters, 3 environments (dev, staging, prod)
- **Tech stacks:** Mixed — 60% Python/Go, 30% Node, 10% other; all containerized
- **Monitoring:** Fragmented — Datadog, CloudWatch, Grafana across teams
- **Platform team size:** 5 engineers
- **Timeline:** 6-month MVP
- **Budget:** Modest; prefer open-source, paid tools only with clear ROI

---

## My Architecture Decisions

### Component 1: Faster CI

- **Docker layer caching** to avoid rebuilding unchanged layers
- **Parallelized CI steps** — unit tests and Docker builds run concurrently
- **Build-once-deploy-everywhere** — single image built once, promoted across dev → staging → prod with different configs (no redundant builds)

### Component 2: Deployment Pipeline (GitOps)

- **ArgoCD** watches IaC repo for image tag changes
- CI builds image → updates image tag in IaC repo → ArgoCD detects change → auto-deploys to dev/staging
- **Production requires manual approval** — configurable approvers per team (team lead, on-call engineer, platform team member)
- **One-click rollback** to previous image version

### Component 3: Self-Service Infrastructure Provisioning

- **Web UI with forms** — developers fill out what they need (database type, size, environment)
- Submission triggers a **workflow engine** (Temporal / Airflow / Argo Workflows) to provision via IaC (Terraform)
- No SRE involvement needed for standard requests

### Component 4: Centralized Observability

- **OpenTelemetry sidecars** injected into every service to collect logs, metrics, traces
- Data flows to platform team's centralized backend
- **Unified Grafana UI** for all teams to query logs, metrics, traces in one place
- Solves monitoring fragmentation — teams no longer need separate Datadog/CloudWatch/Grafana setups

### Rollback Strategy

- **Prevent rather than cure** — enforce backward-compatible database migrations (tested with Alembic in staging)
- Image rollback is safe because DB schema is always backward-compatible
- Rollback scripts tested in dev/staging before prod deployment

### User Interface

- **Two main workflows in the web UI:**
  1. **Deployment page** — view image tags, select version, trigger deployment, approve prod releases
  2. **Infrastructure provisioning page** — fill forms, submit, IDP provisions automatically

---

## Risks Identified

1. **Log format fragmentation** — teams log differently (JSON vs plain text); need to standardize log format as a prerequisite before centralized observability works well
2. **OpenTelemetry sidecar resource contention** — high log/metric volume could overwhelm sidecars; mitigate with batching/buffering (Fluent Bit pattern) and sampling strategies

---

## Scoring Feedback

### What Went Well

- Grounded design in **real problems** from 17LIVE (slow monolith builds, SRE bottlenecks)
- Solutions were pragmatic and production-grade: Docker caching, ArgoCD, OpenTelemetry, Temporal
- Good risk thinking — identified log fragmentation and sidecar resource contention
- Correct instinct on **build-once-deploy-everywhere** and **manual approval for prod**

### What Needs Work

1. **Framework discipline** — jumped to solutions without fully laying out constraints first; should be: clarify → constrain → alternatives → tradeoffs → deep dive
2. **Scope clarity** — MVP tries to solve CI speed + self-service infra + observability + rollback simultaneously; for a 5-person team in 6 months, need to prioritize which is Phase 1 vs Phase 2
3. **Alternatives and tradeoffs** — didn't explore alternatives (e.g., why OpenTelemetry over mandating Datadog for all teams? Why ArgoCD over Flux?)
4. **Deployment UI details** — web UI workflow was vague; how does approval work in practice? Slack notification? Email? Button in UI?

### Key Gap for Staff Level

> Staff level requires showing **HOW you evaluated alternatives**. Example: "I considered Datadog vs OpenTelemetry. Datadog is managed, lower maintenance for a 5-person team. But OpenTelemetry is vendor-agnostic and aligns with our open-source budget constraint. Given our team can handle the operational overhead, OTel is the better long-term choice."

---

## Recommendations for Session 2

- Pick a **stretch topic** — notification system, rate limiter, or payment processing
- Focus on **constraint-first discipline**: state all constraints before touching architecture
- Practice **alternatives + tradeoffs** for every major component (at least 2 options, explain why you picked one)
- Time yourself: 5 min clarify, 3 min constraints, 10 min architecture, 5 min deep dive, 5 min scaling/tradeoffs

---

## Claude Code Prompt

Use the prompt below in Claude Code to generate future session summaries in the same format:

```text
You are a system design interview session summarizer for Jerry Chen, a Staff Platform Engineer preparing for interviews at European tech companies.

Given a system design practice session transcript, produce a markdown summary with exactly these sections:

# System Design Practice — Week [X], Session [Y]
## Topic: [topic name]
- Date, Score (out of 10), Phase (Introductory/Intermediate/Advanced)

## Definition of [Topic]
- What it is, why companies build/need it
- Key success metrics (before/after table)

## Scenario
- The exact question asked
- Clarified constraints (bullet list)

## My Architecture Decisions
- Each component as a subsection with:
  - What it does
  - Key technology choices
  - Why this approach

## Risks Identified
- Numbered list of risks and mitigations

## Scoring Feedback
### What Went Well
### What Needs Work
### Key Gap for Staff Level

## Recommendations for Next Session
- Specific topics and skills to practice

Rules:
- Use Jerry's real tech stack: Kubernetes/EKS, Terraform, OpenTelemetry, ArgoCD, Python, Go, gRPC, Helm, Istio
- Reference his real experience at 17LIVE (monolith-to-microservices, 50K-150K DAU) and Symphox (6 EKS clusters, IDP zero-to-one)
- Score using the rubric: 1=Junior, 2=Mid, 3=Senior, 4=Staff, 5=Principal
- Always note the Staff-level gap: alternatives evaluation and decision framework
- Keep language direct and actionable
```
