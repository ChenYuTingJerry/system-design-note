# Grammar Practice — System Design Interview Patterns

Based on Week 2, Session 1 transcription analysis.

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

**Pattern:** `I would + verb + object + reason`

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

**Problem:** Stating a tool choice without completing the reason. This is also the Staff-level gap — you need to finish the "because" clause.

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

**Pattern:** `I have done this before at [company] by + verb-ing + what you did`

---

## 7. Pronunciation reminders

Keep drilling these every session:

| Word | Correct | Common mistake |
|------|---------|---------------|
| Symphox | SIM-fox | Saint Fox / Same Fox |
| 17LIVE | seventeen live | seventeen life |
| EKS | E-K-S | EK disasters / a-gas |
| tech stack | tech stack | tax stake |
| depth | depth | death / path |

---

## Daily Drill

Pick 3 sentences from above each morning and say them out loud 5 times. Focus on finishing the sentence completely before starting the next one. The biggest pattern to break: **trailing off mid-sentence.** Commit to landing every sentence with a clear ending.
