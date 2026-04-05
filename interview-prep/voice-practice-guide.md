# Voice Practice Guide — System Design Interviews with Claude.ai Mobile

## Why Voice Practice Matters

- Typing answers is not the same as speaking them — fluency on a keyboard does not transfer to fluency out loud
- Real interviews require structured verbal delivery under time pressure
- Speaking reveals gaps that typing hides: filler words, incomplete reasoning, missing trade-offs
- Interviewers evaluate how you communicate, not just what you know
- Practicing out loud builds the muscle memory needed to stay calm and structured in a live setting

## Setup

1. Install the **Claude.ai mobile app** (iOS or Android)
2. Open a new conversation
3. Tap the **voice/microphone icon** to enter voice mode
4. Use a quiet environment — background noise breaks flow
5. Sit at a desk, not a couch — posture affects delivery
6. Keep your phone propped up at eye level if possible (simulates a video call)

## The Kickoff Prompt

Copy and paste this into a new Claude.ai conversation, then switch to voice mode:

```
Act as a staff-level system design interviewer at Uber.
Interview me for a Staff Software Engineer role.

Rules:
- Ask one question or prompt at a time
- Wait for my full answer before responding
- Give me brief feedback after each answer
  (what I covered well, what I missed)
- Then move to the next stage
- Hold me to staff+ level expectations
- Push back if my answer is vague or missing trade-offs

Start with: "Let's design a ride-sharing service like Uber.
Where would you like to start?"
```

## How to Rotate Case Studies

The same prompt works for any problem — just change the last line. Examples:

- `"Let's design a ticketing system like Ticketmaster."`
- `"Let's design a messaging system like WhatsApp."`
- `"Let's design a URL shortener like Bit.ly."`
- `"Let's design a file storage system like Dropbox."`
- `"Let's design a social feed like Facebook News Feed."`

Swap the company name, swap the last line, and the rest of the prompt stays identical.

## End-of-Session Feedback Prompt

After finishing a practice round, paste this to get a structured debrief:

```
Give me a summary of this session:
- What I covered well
- What I consistently missed or was vague about
- Which areas to prioritize before my next session
- My overall readiness: Mid / Senior / Staff+
```

Save the output somewhere you can review it before your next session.

## Tips for Effective Voice Practice

- **Don't look at notes while speaking** — the goal is retrieval, not recognition
- **If you blank, say it out loud:** "Let me think for a second" — interviewers expect this and it buys you time
- **State trade-offs explicitly** — don't wait to be asked ("The trade-off here is consistency vs latency...")
- **Name specific technologies:** Redis, Kafka, Temporal, DynamoDB — not "some kind of cache" or "a message queue"
- **Flag problems proactively** before the interviewer asks ("One concern with this approach is...")
- **Time-box yourself:** aim for 2-3 minutes per answer, then pause for feedback
- **Record yourself** occasionally and listen back — you will catch patterns you don't notice in the moment

## HTML Sim vs Voice Mode

| Dimension | HTML Sim | Voice Mode |
|---|---|---|
| Format | Click-through structured flow | Natural back-and-forth conversation |
| Timer | Built-in strict countdown | Self-managed |
| Feedback | Score at the end | Real-time pushback on vague answers |
| Connectivity | Works offline | Requires internet |
| Best for | Structural warm-up, solo reps | Simulating real interview conditions |
| Difficulty | Lower — prompts guide you | Higher — you drive the conversation |

**Recommended order:**

1. **Study the .md case study note** — understand the system deeply
2. **Warm up with HTML sim** — internalize the structure and key points
3. **Practice with voice mode** — simulate real conditions with live feedback

## Weekly Study Rhythm

| Day | Activity |
|---|---|
| **Mon–Fri** | Discuss new topics in Claude.ai desktop → document via Claude Code after each session |
| **Sat** | Voice practice on 2–3 case studies (Claude.ai mobile) → request end-of-session feedback summary |
| **Sun** | Review GitHub notes for weak areas identified during Saturday's voice session → prioritize topics for next week |

## Case Studies Checklist

Check off each one after completing a voice practice session:

- [ ] Uber
- [ ] Bit.ly
- [ ] Dropbox
- [ ] Ticketmaster
- [ ] WhatsApp
- [ ] FB News Feed
- [ ] Tinder
- [ ] LeetCode
- [ ] Yelp
- [ ] Strava
- [ ] Rate Limiter
- [ ] Online Auction
- [ ] FB Live Comments
- [ ] FB Post Search
- [ ] Price Tracking Service
- [ ] Instagram
- [ ] YouTube Top K
- [ ] Robinhood
- [ ] Local Delivery Service
- [ ] News Aggregator

## Key Takeaways

- **Speaking is a separate skill from knowing** — you must practice it independently
- **Voice mode with Claude.ai mobile is the closest simulation** to a real interview without a human partner
- **Structure matters more than completeness** — a clear, well-organized answer that covers 80% beats a rambling answer that covers 100%
- **Use the HTML sim to learn the material, use voice mode to learn to deliver it**
- **Weekly feedback loops compound** — Saturday voice sessions surface gaps, Sunday reviews close them, weekday study fills them
- **The kickoff prompt is reusable** — one template covers every case study in the checklist
