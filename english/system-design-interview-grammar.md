# Grammar Practice — System Design Interview Patterns

Based on Week 2, Session 1 transcription analysis + Senior/Staff speaking patterns.

---

## 1. "Does each team..." questions

**Problem:** Mixing up structure in "does each team have/use" questions.

**Practice these:**
- "Does each team have a different tech stack?"
- "Does each team use different programming languages?"
- "Does each team use a different monitoring tool?"
- "Does each team have its own deployment process?"

**Pattern:** `Does each team + have/use + a/its own + noun?`

---

## 2. "I will / I would" decision statements

**Problem:** Switching between tenses mid-sentence, or trailing off before finishing the thought.

**Practice these:**
- "I would build a centralized monitoring service."
- "I would use OpenTelemetry as a sidecar for each service."
- "I would go with a six-month timeline for the MVP."
- "I would adopt a GitOps approach using ArgoCD."

**Pattern:** `I would + verb + object (+ reason)`

---

## 3. "It will trigger..." event chains

**Problem:** Losing sentence structure when describing event flows.

**Practice these:**
- "After the CI process is done, it will trigger the CD process automatically."
- "Once the image is built, the system will send an event to the platform team's management service."
- "After submitting the form, it will automatically trigger the IaC process to provision the database."
- "When the monitoring system finds an issue, it will send a notification to the target team."

**Pattern:** `After/Once/When + event, + it will + action`

---

## 4. "They can... so we can..." purpose chains

**Problem:** Tangling up "they/we/it" subjects when explaining who does what.

**Practice these:**
- "Developers fill out the form, and then the system provisions the infrastructure automatically."
- "Each team provides their list of approvers, so the system knows who can approve production deployments."
- "Teams push their code to GitHub, and the CI process builds the image."
- "The OpenTelemetry sidecar collects the logs, so we can provide a unified query interface."

**Pattern:** `[Subject A] does X, so [Subject B] can do Y`

---

## 5. "I would use X because..." justification statements

**Problem:** Stating a tool choice without completing the reason.

**Practice these:**
- "I would use OpenTelemetry because it is vendor-agnostic and fits our open-source budget."
- "I would use ArgoCD because it integrates well with Kubernetes and supports GitOps natively."
- "I would use Docker layer caching because it avoids rebuilding unchanged layers."
- "I would choose Grafana over Datadog because it is open-source and reduces licensing cost."

**Pattern:** `I would use/choose + tool + because + reason`

---

## 6. "I have done this before by..." experience references

**Problem:** Stumbling when connecting past experience to the current design.

**Practice these:**
- "I have done this before at Symphox by building an IDP on EKS from scratch."
- "I have dealt with this at 17LIVE, where the monolith CI was extremely slow."
- "At 17LIVE, we solved this by using the strangler fig pattern."
- "At Symphox, I built a self-service data platform that reduced SRE toil."

**Pattern:** `I have done this before at [company] by + verb-ing`

---

# 🆕 7. "I noticed... so I stepped in..." ownership pattern (Staff)

**Problem:** Sounding passive or waiting for direction.

**Practice these:**
- "I noticed the monolith was becoming a bottleneck."
- "There wasn’t a clear owner driving the migration."
- "So I stepped in and scoped the problem."

**Pattern:**
`I noticed that...`
`There wasn’t really...`
`So I stepped in and...`

---

# 🆕 8. "There were a few constraints..." framing pattern (Staff)

**Problem:** Jumping into solutions without context.

**Practice these:**
- "There were a few key constraints."
- "First, we had high traffic and strict availability requirements."
- "Second, we couldn’t pause feature delivery."
- "Third, the team was very small."

**Pattern:**
`There were a few key constraints.`
`First...`
`Second...`
`Third...`
`Also...`

---

# 🆕 9. "Given that... I looked at options..." decision pattern (Staff)

**Problem:** Not showing decision-making process.

**Practice these:**
- "Given that, I looked at a few options."
- "A would be cleaner, but too risky."
- "B would delay value too much."
- "So we chose X as the best trade-off."

**Pattern:**
`Given that...`
`I looked at a few options.`
`A..., but...`
`B..., but...`
`So we chose X as the best trade-off.`

---

# 🆕 10. "Basically, we..." explanation pattern (Fluency)

**Problem:** Overcomplicated sentences when explaining systems.

**Practice these:**
- "Basically, we extracted services one by one."
- "We ran them alongside the monolith."
- "And then we removed the old code."

**Pattern:**
`Basically, we...`
`We...`
`And then we...`

---

# 🆕 11. "That helped us..." impact pattern

**Problem:** Describing actions without impact.

**Practice these:**
- "That helped us keep the risk low."
- "It also meant we could adjust the design."
- "It allowed us to keep delivering features."

**Pattern:**
`That helped us...`
`It also meant we could...`
`It allowed us to...`

---

# 🆕 12. "We made it a rule..." system thinking pattern

**Problem:** Not showing long-term mechanisms.

**Practice these:**
- "We made it a rule that all new features go into the new services."
- "We enforced a standard deployment process."

**Pattern:**
`We made it a rule that...`

---

# 🆕 13. "If we had... but given..." trade-off depth (Staff)

**Problem:** Decisions sound absolute instead of contextual.

**Practice these:**
- "If we had more engineers, a big-bang rewrite might have worked."
- "But given our constraints, we chose strangler fig."

**Pattern:**
`If we had..., then...`
`But given..., we...`

---

# 🆕 14. "X made the most sense..." soft decision pattern

**Problem:** Sounding too absolute.

**Practice these:**
- "Strangler fig made the most sense."
- "This felt like the best trade-off."

**Pattern:**
`X made the most sense.`
`X felt like the best trade-off.`

---

# 🆕 15. "Looking back..." reflection pattern (Staff signal)

**Problem:** Ending answers too abruptly.

**Practice these:**
- "Looking back, I think the key was aligning with constraints."
- "Looking back, the decision worked because we prioritized risk."

**Pattern:**
`Looking back...`
`I think the key was...`

---

# 🆕 16. Natural speaking fillers (Fluency)

**Problem:** Sounding robotic.

**Practice these:**
- "So..."
- "Basically..."
- "And yeah..."
- "Kind of..."

**Pattern:**
Use naturally between ideas to create pause and flow.

---

## 17. Pronunciation reminders

Keep drilling these every session:

| Word | Correct | Common mistake |
|------|---------|---------------|
| Symphox | SIM-fox | Saint Fox / Same Fox |
| 17LIVE | seventeen live | seventeen life |
| EKS | E-K-S | EK disasters |
| tech stack | tech stack | tax stake |
| depth | depth | death / path |

---

## Daily Drill

Pick 3–5 patterns and say them out loud 5 times.

Focus on:
- finishing the sentence clearly
- adding natural pauses
- not rushing

The biggest pattern to break:
👉 **trailing off mid-sentence**

The most important skill to build:
👉 **speaking in patterns, not memorizing scripts**