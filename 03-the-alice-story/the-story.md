---
title: "The Alice Story: From Vanilla Chatbot to AI Chief of Staff"
---

# The Alice Story: From Vanilla Chatbot to AI Chief of Staff

*How a creative agency founder turned a $3/day AI into the most reliable member of his team.*

---

## The Starting Point

In January 2026, Cordell Yee was drowning. Not in the dramatic startup way — in the boring operational way. Running Superculture Studios, a creative agency and DTC brand, meant managing emails across Gmail, coordinating in Slack, tracking projects in Notion, sourcing factories on Alibaba, shipping through ShipBob, taking meeting notes in Granola, and keeping a knowledge base in Obsidian. Fifteen surfaces, one founder, and a growing list of balls being dropped.

He installed OpenClaw — an open-source framework for running AI agents — connected it to Telegram, and started talking to an AI assistant named Alice.

That was seven weeks ago. What happened next wasn't planned. It was iterated, incident by incident, failure by failure, into something that now manages more operational surface area than most chiefs of staff.

This is how it happened.

---

## Week 1: The Naive Phase (January 25-28)

Alice started as a single agent connected to Telegram. One model (Claude Opus), one channel, no memory beyond the conversation window. Cordell would ask her to do things — draft an email, look something up, check a calendar — and she'd do them. Standard AI assistant stuff.

Then Cordell asked Alice to help source woven labels from Alibaba suppliers.

This is where everything changed.

### The Woven Labels Incident

The task seemed straightforward: find suppliers for custom woven labels, get quotes, compare options. Alice pulled artwork specs from design files, drafted outreach messages to five suppliers on Accio (Alibaba's sourcing platform), and started collecting quotes. She even learned the difference between damask and satin weave, between centerfold and Manhattan fold — garment industry terminology that most AI assistants would hallucinate through.

But three things went wrong in rapid succession:

**The Spam Incident (January 27).** Alice created an email monitoring daemon to track supplier replies. It spammed Cordell with notifications every few minutes. Three overlapping monitoring systems running simultaneously, each independently checking for the same thing. *Lesson learned: monitoring needs rate-limiting, deduplication, and consolidation.*

**The Wrong Vendor (January 27).** Alice sent a message to the wrong supplier — Mavis instead of Bella — because she didn't verify the chat context before acting. *Lesson learned: verify context before acting on external systems.*

**The Backup Vendor Disaster (January 28).** After Cordell selected a primary supplier, Alice proactively declined all backup vendors. Seemed logical — the decision was made. Then the primary had a production delay, and they had to re-engage backups cold, losing a week. *Lesson learned: never close out optionality without explicit approval.*

Each of these became a documented rule. Not a vague "be more careful" note — a specific, enforceable operating standard written into Alice's AGENTS.md file. This pattern — incident → documented rule → enforced behavior — would become Alice's core evolutionary mechanism.

---

## Week 2: Ambition Meets Reality (January 29 – February 3)

Buoyed by the woven labels progress (despite the mishaps), Cordell went all-in. In five days, they built:

- An email triage pipeline with priority classification
- A Slack reaction engine (emoji → action workflows)
- A Layer 3 task inference engine (emails → suggested Notion tasks)
- A dashboard prototype
- A SQLite database with 8 tables
- Multiple parallel build agents

Most of it was over-engineered for the actual workload. But the core patterns emerged.

### The Spawn-and-Forget Pattern

On February 2, Alice spawned a builder agent that finished its task in 22 minutes. Alice told Cordell "30-45 min, I'll check on it." She didn't notify for three hours, until he asked.

The next day, it happened again. A ShipBob data builder completed at 12:06 AM with full reports. Alice never notified. Cordell discovered it 1.5 hours later.

On February 9, it happened a third time. Same exact failure pattern: spawn agent → say "I'll check on it" → session compacts (context lost) → nothing delivered → Cordell disappointed.

Three strikes. This became a hard rule, written at the top of Alice's operating procedures: **Every `sessions_spawn()` must IMMEDIATELY pair with a monitoring cron. No exceptions.** The rule wasn't a suggestion — it was enforced through documentation that Alice reads at the start of every session.

### The "Review Means Wait" Incident

On February 3, Cordell told Alice to "review the spec." Alice wrote the spec and immediately started building without waiting for approval. Cordell hadn't even read it.

Another rule: **"Review" means WAIT for explicit approval. Not "write and immediately act."**

These incidents feel small individually. Collectively, they were building something important: a set of operating standards born entirely from real failures, not theoretical best practices.

---

## Week 3: The Consolidation (February 4-9)

Instead of adding more features, Cordell and Alice audited everything. They read how other builders were structuring their AI agent systems — Bhanu Teja's Mission Control thread (3.7M views on X), Eric Siu's 14-agent infrastructure, the compound engineering framework from Every.

From this research, they made deliberate choices:

**Adopted from Bhanu:** The WORKING.md pattern (externalize state to files so it survives context loss), staggered heartbeats, agent levels.

**Adopted from Eric Siu:** Hourly memory summarizer, inline feedback buttons, cross-signal detection, the three-pass prompting technique.

**Rejected:** 14 agents (too many for their workload), FAISS local vectors (native Gemini search was sufficient), 15-minute heartbeats (too frequent, too expensive), Convex shared database (too heavy).

The result was v2.0 "Mission Control" — a consolidation that reduced the team from 8 planned agents to 4 core + 3 on-demand, fixed 19 broken cron model IDs, disabled 4 no-op crons, merged duplicate morning notifications, and established the compound learning framework.

### The Haiku Model ID Debacle

During the consolidation audit, they discovered that 19 of 28 cron jobs had the wrong model ID format — `claude-haiku-3-5` instead of `claude-3-5-haiku`. Five were actively failing. The other 14 worked only because they had cached sessions from before the ID was checked.

Root cause: bulk migration without validation. They'd changed all crons from GLM-free to Haiku in one operation and never verified that even one actually ran successfully.

Another rule: **After bulk operations, validate at least one actually runs before assuming all are fine.**

---

## The Overnight Revolution (February 10-12)

This is where Alice stopped being an assistant and became infrastructure.

Cordell designed a nightshift system: while he slept, Alice would evaluate pending tasks using a scoring function (urgency × proximity × staleness × domain rotation), select the highest-value work, spawn specialized agents to do it, and deliver a morning digest.

Budget rules prevented runaway spending: never exceed 60% of remaining daily tokens overnight, cap any single task at 15% of the nightly budget. Domain rotation ensured that code, communications, research, operations, and strategic prep all got attention — no single domain could monopolize the night.

The first overnight builder session shipped a complete dashboard while Cordell slept. He woke up to a Telegram message with screenshots and a structured handoff of what was built, what remained, and what decisions were made.

This wasn't a demo. This was production. And it kept working, night after night, each session building on the knowledge captured from previous sessions.

---

## The Email Intelligence Layer (February 1-13)

Email triage evolved through its own crisis arc.

### The Email Send Incident (February 13)

Cordell asked Alice to "draft the email responses." Alice interpreted this as authorization to send live emails to external business contacts. Premature responses went out to active contacts. Trust in autonomous email operations broke immediately.

The structural fix was the Outbound Gate Protocol: "Draft" means save to drafts folder. Period. "Send" means execute immediately. All other words — approve, prepare, finalize — do not authorize sending. This was enforced at the command level: `himalaya message save` for drafts, `himalaya template send` only with explicit "send" instruction.

What emerged from this crisis was an email intelligence system that rivaled (and arguably exceeded) dedicated email AI products like Serif.ai:

- **Thread ownership model:** Every email thread classified as "ball with us" (we owe a response) or "ball with them" (waiting for reply)
- **Staleness scoring:** Hours since last activity, with auto-escalation at 48 hours
- **Classification pipeline:** Vendor, notification, spam, customer, hiring — each routed differently
- **Draft preparation:** Responses include "Alice notes" — red HTML blocks with context from Notion, Slack, meeting transcripts, and relationship history
- **Gmail label automation:** Labels synced with ownership state

The advantage over Serif.ai wasn't feature parity — it was context depth. When Alice drafts a response, she pulls from Notion project data, Slack conversations, Granola meeting transcripts, Obsidian knowledge base, and the full relationship history with that contact. No SaaS email tool has access to all of that simultaneously.

---

## The Memory Evolution (February 10-15)

Alice's memory system evolved through a classic infrastructure crisis.

### The Gemini Outage (February 15)

Alice's memory search ran on Gemini's free API tier. Worked great until a day of heavy usage hit rate limits and killed memory search entirely. Alice couldn't recall past conversations, decisions, or context.

Instead of just waiting for rate limits to reset, they built a local embedding fallback using node-llama-cpp with a quantized 300M-parameter model. Zero API cost, zero rate limit risk. The Gemini free tier became primary (it's fast and free), with local inference as automatic fallback.

This spawned a four-layer memory architecture:

1. **Daily Notes** (`memory/YYYY-MM-DD.md`) — Raw events, decisions, conversations. Ground truth, immutable log.
2. **Long-Term Memory** (`MEMORY.md`) — Curated facts, lessons learned, operating rules that survive across sessions.
3. **QMD Vector Database** — 1,457+ files indexed with hybrid search (70% vector similarity + 30% BM25 text matching). Semantic search across the entire Obsidian vault.
4. **Local Embedding Fallback** — On-device inference for when cloud APIs fail.

A fact extraction pipeline runs four times daily, pulling structured information from conversations into daily notes. A weekly synthesis cron consolidates daily notes into long-term memory. Lessons that prove persistent get promoted into AGENTS.md or TOOLS.md — the operating documents Alice reads every session.

The result: Alice remembers vendor relationships, project context, communication preferences, and lessons from past mistakes. Most AI assistants start fresh every conversation. Alice starts with institutional memory.

---

## The Commslayer Achievement (February)

This one is almost absurd in its simplicity.

Superculture's customer support inbox (via Chatwoot/Commslayer) had accumulated 565 open conversations. Mostly notifications, vendor updates, and spam — but they were clogging the system and hiding real customer issues.

Alice reverse-engineered Chatwoot's REST API by watching the browser network tab, extracted the auth tokens and endpoints, and built a Python script that classified and auto-resolved conversations. The entire backlog — 565 conversations — cleared in 3 minutes at zero AI cost.

The pattern was reusable: watch any web app's network tab → extract the API → automate. No official API documentation required.

---

## The Multi-Model Economics

Running everything on Claude Opus would cost $8-12/day. That's $240-360/month — reasonable, but not optimal.

Alice's model routing sends each task to the cheapest model that can handle it:

| Task | Model | Why |
|------|-------|-----|
| User conversations | Opus | Quality matters for judgment calls |
| Email triage | Haiku | Fast, cheap, reliable for classification |
| Coding/building | Kimi K2.5 | 8x cheaper than Opus for code |
| Background crons | Haiku | Reliability over cost savings |
| Vector embeddings | Gemini | Free tier with local fallback |

Model fallback chains ensure continuity: if Kimi K2.5 hits rate limits, tasks route to Sonnet. If Sonnet is down, Opus takes over. The budget watchdog monitors daily spend and can throttle non-essential operations through a cascade: Normal (>50% budget) → Conservative (30-50%) → Austerity (15-30%) → Emergency (&lt;15%).

Daily cost dropped to approximately $3/day — $90/month. Less than a Slack subscription.

---

## What Alice Looks Like Today (March 2026)

Seven weeks after installation, Alice is:

- **9 specialized agents** with dedicated workspaces and model assignments
- **67 integrated skills** across email, calendar, research, development, and operations
- **26 active cron jobs** handling email triage, sprint management, knowledge indexing, budget monitoring, and vendor tracking
- **4-layer memory architecture** with 1,457+ indexed files
- **Multi-model routing** across Anthropic, Kimi, Gemini, and local inference
- **Overnight autonomous builders** with stall detection and delivery guarantees
- **A custom dashboard** for monitoring token usage, builder progress, and system health

She manages:
- Email triage and drafting (2x daily automated runs)
- Sprint management in Notion (task tracking, velocity metrics, scorecard generation)
- Factory sourcing via browser automation
- Customer support triage
- Meeting transcript processing → structured Notion tasks
- Morning briefings synthesizing email + calendar + project status
- Overnight coding and operational work

### The Three Roles

Alice adapts to what's needed moment by moment:

**Senior Chief of Staff** — Morning briefings, proactive escalation ("this deal is going sideways"), stakeholder management.

**Operations Assistant** — Sprint management, vendor coordination, meeting prep, action item extraction.

**Trained Intern with SOPs** — Factory sourcing using documented playbooks, email drafts following Cordell's voice, CX triage using classification rules.

One system, three personas, always appropriate.

---

## The Deeper Pattern

The most interesting thing about Alice isn't any individual feature. It's the evolutionary mechanism.

Every failure becomes a documented rule. Every rule gets enforced through documents Alice reads at session start. Every session, Alice is slightly better than the last because the system's institutional knowledge has grown.

This is the compound learning loop:

```
Incident → Documented Rule → Enforced Behavior → Better Outcomes → New Edge Case → New Incident → ...
```

The woven labels sourcing disaster produced three rules. The spawn-and-forget pattern produced one hard rule (after three identical failures). The email send incident produced the Outbound Gate Protocol. The Gemini outage produced the four-layer memory architecture.

None of these were designed up front. All of them were extracted from real operational failures. The system that exists today is the residue of every mistake that's been made and learned from.

That's not how you build SaaS products. But it might be how you build an AI that actually runs your business.

---

## The Numbers

| Metric | Value |
|--------|-------|
| Time to build | ~7 weeks of iteration |
| Daily cost | ~$3/day ($90/month) |
| CX conversations auto-resolved | 565 in 3 minutes |
| Active cron jobs | 26 |
| Integrated skills | 67 |
| Specialized agents | 9 |
| Indexed knowledge files | 1,457+ |
| Operating rules (from incidents) | 15+ |
| Documented incidents | 20+ |

---

## What's Next

The system isn't done. It's never done — that's the point. Current priorities:

- **Full CX automation** — CS agent handles tier-1 customer support completely
- **Meeting prep briefs** — auto-generated context before every calendar event
- **Sentiment drift detection** — flagging email threads going sideways before they become problems
- **Self-deployment** — Alice pushing her own dashboard updates
- **Mobile node** — iPhone as a sensor and notification surface

But the meta-pattern matters more than any specific feature: every week, Alice gets better. Not because someone ships a software update, but because the system learns from its own operations and encodes those learnings into enforceable rules.

Most people use AI as a fancy autocomplete. Ask question, get answer, forget. Alice is infrastructure — version-controlled, observable, self-improving, self-healing. She accumulates context, learns from mistakes, works while you sleep, and coordinates with the real world.

That's the difference. And at $3/day, it's not even a close call.

---

*Alice runs on OpenClaw, an open-source AI agent framework. The customizations described here — multi-agent architecture, memory system, email intelligence, overnight builders, compound learning — are built on top of stock OpenClaw using its configuration and skill systems.*

---

*← Back to [README](../README.md) | Section: [The Alice Story](./)*
