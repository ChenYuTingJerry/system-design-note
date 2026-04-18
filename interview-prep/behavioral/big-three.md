# Big Three Interview Questions
> Jerry Chen — Staff Platform Engineer

## Q1: Tell Me About Yourself (TMAY)

### Keyword Card
OPENING
→ help teams: faster + independent
→ remove platform bottlenecks

SYMPHOX
→ no unified platform
→ EC2 + Lambda, manual, blocked
→ proposed K8s on AWS
→ built from scratch: 6 EKS clusters
→ deploy + monitor + manage independently
→ no SRE handoff
→ self-service data platform
→ 40% faster → just ship

17LIVE
→ large monolith → scaling bottleneck
→ led migration → 4-5 teams
→ hardest: not tech → alignment
→ designed solution → demoed to VP
→ cross-team meeting → what to migrate first
→ evolutionary approach → selectively
→ no big bang → no production disasters

EM YEAR
→ another team needed manager
→ director asked → I stepped in
→ stayed hands-on: architecture + incidents + platform
→ taught me: real engineer friction
→ real energy: technical depth not management
→ came back to IC

CLOSING
→ relocating Netherlands → wife starting school
→ excited to bring this to your team
→ engineers: products not infrastructure
→ happy to dive deeper

### Full Answer
I help engineering teams move faster and become more independent — by getting the platform bottlenecks out of their way.

At Symphox, engineering teams had no unified platform. Services were scattered across EC2 and Lambda, deployments were manual, and teams were constantly blocked waiting for help. So I proposed moving everything to Kubernetes on AWS and built the entire platform from scratch — six EKS clusters — where teams could deploy, monitor, and manage their own services independently, no SRE handoff required. I also built a self-service data platform so the data team could own their own pipelines. Deployments became 40 percent faster, and teams could finally just ship without asking for help.

Before that at 17LIVE, the company was held back by a large monolith. I led the migration to microservices across four or five teams. The hardest part wasn't the technology — it was alignment. I designed the solution, demoed it to the VP, and ran a cross-team meeting to decide what to migrate first. We took an evolutionary approach — migrate selectively, reduce risk, and made sure every new feature was built on the new architecture. No big bang, no production disasters.

Last year, another team needed a new manager and my director asked if I'd step in. I took the role but stayed hands-on the whole time — architecture, incidents, platform work. That year taught me what engineers actually struggle with every day, from both sides. But I realized my real energy is in technical depth, not people management. So I came back to full-time IC work.

My family and I are relocating to the Netherlands — my wife is starting school there. I'm excited to bring all of that to your team and help your engineers focus on building products, not fighting infrastructure. Happy to dive into any of these.

### Target Length
90–120 seconds

### Signals
- Scope: 6 EKS clusters, VP-level alignment
- Ownership: proposed EKS from zero
- Leadership: EM year, cross-team coordination
- Growth: EM taught real engineer friction

---

## Q2: Tell Me About Your Favorite Project

### Keyword Card
CONTEXT
→ joined Symphox → infrastructure chaos
→ EC2 manual + Lambda no version control
→ no platform ownership → knowledge siloed
→ every new project = reinventing the wheel

ACTIONS
→ proposed Kubernetes on AWS
→ chose EKS (AWS already in use)
→ built first cluster myself
→ migrated all running projects
→ Terraform IaC so team can maintain together
→ Azure DevOps for CI/CD standardization
→ self-service data platform

RESULTS
→ standardized entire CI/CD process
→ any engineer can deploy any service same way
→ deployment cycles: 40% faster
→ observability onboarding: 5x faster
→ knowledge no longer siloed

LEARNINGS
→ real value of platform = making knowledge accessible
→ when team can self-serve → they scale
→ removing bottlenecks > adding features

### Full Answer
When I joined Symphox, the infrastructure was a mess. Different teams were deploying manually to EC2, others were writing code directly in Lambda with no version control. There was no standardization at all — knowledge was trapped in individual people's heads, and every new project meant starting from scratch.

So I proposed moving everything to Kubernetes. AWS was already our cloud provider, so I chose EKS. I built the first cluster myself, migrated all running projects onto it, and introduced Terraform so the entire team could maintain the infrastructure as code — not just me. I also standardized CI/CD through Azure DevOps so every service followed the same deployment process.

The impact was significant. Deployment cycles dropped 40 percent. Observability onboarding became five times faster. Any engineer could now deploy any service the same way — the knowledge that used to be trapped in one person's head became accessible to everyone.

What I learned is that the real value of a platform isn't the technology. It's making knowledge accessible so teams can self-serve and scale without bottlenecks. That's the kind of platform work I want to do in my next role.

### Target Length
90–120 seconds

### Signals
- Scope: full platform stack, multiple teams
- Ownership: proposed and built from zero
- Ambiguity: no playbook, designed the approach
- Communication: taught the team, made knowledge accessible

---

## Q3: Tell Me About a Conflict

### Keyword Card
CONTEXT
→ billing team struggling 2 years
→ tech lead (Jarvis): people manager not tech leader
→ hiding info → P0 monthly → whole team overtime
→ PM had no visibility → stakeholders couldn't decide
→ I proposed intervention to supervisor + GM
→ stepped in as Staff → became EM 2 months later

ACTIONS
→ didn't use authority as hammer
→ introduced team norms: transparent decisions, Confluence
→ feature owner model: removed single bottleneck
→ 1-on-1: told him what tech lead actually does
→ gave specific tasks → he said "I'll think about it" → never followed through
→ stalling tactic: hoping priority would drop
→ stopped waiting → adjusted tasks every cycle
→ reported to supervisor regularly
→ turning point: capability gap not behavior
→ changed strategy: moved him to pre-prod quality validation
→ things finally worked

RESULTS
→ P0: monthly → quarterly (75% reduction)
→ overtime stopped
→ PM got visibility
→ delivered reconciliation project on schedule
→ all 4 engineers stayed

LEARNINGS
→ should have acted faster
→ when feedback isn't landing → protect team sooner
→ didn't expect: self-interest over team even after everything made clear
→ recognize that pattern earlier → act faster

### Full Answer

**Context:**
When I joined Symphox as a Staff Engineer, I was collaborating cross-functionally with a billing system team. The team had been struggling for two years — their tech lead was acting as a people manager instead of a technical leader. He was hiding information from stakeholders, the team was firefighting P0 incidents every month, and everyone was constantly working overtime. The PM couldn't get reliable updates, and no one outside could intervene because he controlled all the context.

After observing the situation, I proposed an intervention plan to my supervisor and the GM — I'd step in, break the silo, and fix the team structure. Two months later, I became the EM and he became my direct report.

**Actions:**
Even with formal authority now, I deliberately didn't use it as a hammer. I wanted to fix the root problem, not just escalate.

First, I introduced team norms — all technical decisions had to be documented and discussed in weekly tech reviews, open to everyone. No more private decisions. This immediately broke the information silo.

Second, I introduced a Feature Owner model — any engineer could own a feature end-to-end. This removed him as the single bottleneck.

Third, I used weekly 1-on-1s to tell him clearly what a tech lead actually does — architecture, design, code review — and gave him specific tasks. He would say "I'll think about it and come back to you." But he never did. He was stalling, hoping the priority would drop.

So I stopped waiting. Every cycle, I adjusted the tasks based on what he could and couldn't deliver, and I reported progress to my supervisor regularly.

The turning point came when I realized the gap wasn't just behavior — it was capability. He wasn't avoiding technical work out of stubbornness. He genuinely didn't know how to do it. So I changed strategy completely — I moved him into pre-production quality validation. That's when things started to work.

**Results:**
Within a year, the team completely turned around. P0 incidents dropped from monthly to once a quarter — a 75% reduction. The overtime stopped. The PM finally had visibility. We delivered the billing system reconciliation project on schedule. And all four engineers stayed through the full engagement. Jarvis found a role where he could actually contribute instead of blocking everyone else.

**Learnings:**
Two things. First, I should have acted faster. I spent too long trying to coach someone who wasn't responding. When feedback isn't landing after multiple clear conversations, the right move is to protect the team sooner, not wait longer.

Second, I genuinely didn't expect someone to consistently prioritize their own interests over the team — even after everything was made clear and adjusted multiple times. It taught me to recognize that pattern earlier. Some people won't change until the cost of not changing becomes personal. Now I know — when I see that pattern, I act on it faster.

### Target Length
2.5–3 minutes

### Signals
- Conflict Resolution: handled resistant person directly and professionally
- Influence without authority: no formal authority at first
- Ownership: proposed intervention voluntarily
- Perseverance: one year, multiple strategy adjustments
- Leadership: designed systems, found right role for person
- Growth: honest about acting too slowly

---

## Signal Coverage Map

| Signal | Story |
|---|---|
| Scope | TMAY (6 EKS clusters, VP alignment) |
| Ownership | TMAY + Q2 (proposed from zero) |
| Ambiguity | Q2 + Q3 (no playbook) |
| Perseverance | Q3 (1 year, multiple adjustments) |
| Conflict Resolution | Q3 (Jarvis) |
| Growth | TMAY (EM year) + Q3 (acted too slowly) |
| Communication | TMAY (VP alignment, risk model) |
| Leadership | TMAY + Q3 (system design, role restructuring) |

---

## One Story, Many Questions

| Question | Story to use |
|---|---|
| Tell me about yourself | TMAY |
| Tell me about your favorite project | Q2 Developer platform |
| Tell me about a conflict | Q3 Billing team |
| Tell me about influencing without authority | Q3 Billing team |
| Tell me about delivering difficult feedback | Q3 Billing team |
| Tell me about pushing through a challenge | Q3 Billing team |
| Tell me about leading without a title | Q3 Billing team |
| Tell me about changing your approach | Q3 Billing team |
