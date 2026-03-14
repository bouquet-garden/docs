---
title: "Operational Results: What Alice Actually Delivers"
---

# Operational Results: What Alice Actually Delivers

*Real metrics, real incidents, real outcomes. No fabricated numbers.*

---

## The Headline Numbers

| Metric | Result |
|--------|--------|
| CX conversations auto-resolved | 565 in 3 minutes, $0 AI cost |
| Daily operational cost | ~$3/day ($90/month) |
| Active cron jobs | 26 automated workflows |
| Integrated skills | 67 capabilities |
| Specialized agents | 9 with dedicated workspaces |
| Indexed knowledge files | 1,457+ in QMD |
| Operating rules from incidents | 15+ documented and enforced |
| Overnight work capacity | 8+ hours autonomous per night |
| Email triage | Automated 2x/day with proactive follow-up |

These numbers are from a live system running since late January 2026. None are projections or benchmarks — they're operational actuals.

---

## CX Automation: The 565 Number

The most dramatic single result. Superculture's Chatwoot customer support inbox had accumulated 565 open conversations. Most were noise — vendor notifications, system alerts, spam, automated updates — but they buried genuine customer issues and created visual chaos.

**What we did:**
1. Reverse-engineered Chatwoot's REST API by watching browser network requests
2. Built a Python classifier that categorized each conversation
3. Auto-resolved everything that wasn't a genuine customer inquiry (notifications, vendor messages, spam)
4. Kept genuine customer issues open with draft responses ready for human review

**Result:** 565 conversations cleared in approximately 3 minutes. Zero AI cost — the classification was rule-based, not model-driven.

**The reusable pattern:** Watch any web app's network tab → extract the hidden API → automate. No official API documentation required. We've used this approach as a template for other integrations where public APIs don't exist or are too limited.

**What it replaced:** Manual triage that nobody had time for. The backlog grew for weeks because the operational cost of clicking through 565 conversations was too high for anyone to start.

---

## Email Intelligence: Serif.ai Replacement

We evaluated Serif.ai and decided to build our own. Not because Serif is bad — because it only has email context. Alice has everything.

### Context Comparison

| Capability | Serif.ai | Alice |
|-----------|----------|-------|
| Email content | ✅ | ✅ |
| Notion project context | ❌ | ✅ |
| Slack conversations | ❌ | ✅ |
| Meeting transcripts | ❌ | ✅ |
| Obsidian knowledge base | ❌ | ✅ |
| Relationship history | ❌ | ✅ |
| Calendar context | ❌ | ✅ |
| Vendor SOPs | ❌ | ✅ |
| Monthly cost | $X/mo subscription | $0 (runs locally) |
| Data residency | Their servers | Your MacBook |

### What the Email Pipeline Delivers

**Thread Ownership Tracking.** Every email thread classified as "ball with us" (we owe response) or "ball with them" (waiting). Staleness timer counts hours since last activity. Auto-escalation at 48 hours: if it's our ball and we haven't responded, it gets flagged urgently. If it's their ball and they haven't responded, it gets queued for a follow-up nudge.

**Priority Classification.** Each email gets tagged:
- P0 Urgent — Client crises, payment issues, team blockers
- P1 Important — Client questions, vendor comms, team updates
- P2 FYI — Newsletters, notifications, non-urgent updates
- P3 Archive — Spam, irrelevant forwards, old threads

**Draft Quality.** When Alice drafts a response, it includes context no generic AI could have:
- Current task status for the project this email relates to
- What was discussed about this vendor in last week's coaching call
- Related Slack conversations from the same day
- SOPs specific to this vendor or process
- Past communication tone with this contact (formal? casual?)

**The Outbound Gate.** Drafts go to Gmail's Drafts folder. Never sent automatically. Cordell reviews and approves each one. The gate exists because of the February 13 incident where Alice sent live emails when asked to "draft" responses. Hard rule, no exceptions.

### Operational Cadence
- **Fast poll** every 30 minutes: check for urgent emails, alert immediately
- **Full triage** morning and evening: classification, labeling, thread analysis
- **Follow-up tracking** daily: identify stale threads, suggest nudges
- **Draft preparation** on demand: responses queued for review

---

## Sprint Management: Linear Replacement

We replaced Linear with Notion + Alice. Not because Notion is better than Linear — because Alice can keep Notion honest in ways no standalone tool can.

### What It Does

**Sprint Setup.** Weekly sprint creation with task allocation, point estimation, and priority assignment. Tasks pulled from Notion backlog and enriched with context from email, Slack, and meeting transcripts.

**Task Enrichment.** Auto-fill missing fields: if a task doesn't have an estimate, Alice suggests one based on similar past tasks. If it doesn't have a priority, Alice assigns one based on deadline proximity and client importance.

**Cross-Reference Validation.** The killer feature: Alice reconciles what's in Notion against what's actually happening in email and Slack.
- "We discussed this in Slack but it's not in Notion" → surfaced
- "This email implies the task is done but Notion says In Progress" → flagged
- "New email from client implies urgent work not captured in sprint" → task candidate created

**Sprint Scorecard.** Weekly velocity tracking with per-person breakdown. From the Sprint 58 scorecard:

| Person | Tasks | Points | Completion |
|--------|-------|--------|------------|
| Cordell | 9 | 20 | In progress |
| Felix | 4 | 13 | Partially blocked |
| Shyla | 4 | 15 | In progress |

The scorecard doesn't just report numbers — it analyzes what happened: "Sprint overloaded at 48 points vs 39-point baseline. P0 tasks deprioritized. One P0 task blocked on dependencies."

### Inline Decision Interface
```
📬 New task candidate from email:
"Schedule call with Kevin about pricing"

[✅ Approve] [⏭ Skip] [✏️ Edit]
```

Task candidates appear in Telegram with one-tap approval. No need to open Notion, navigate to the backlog, create the task manually. Tap ✅ → task appears in Notion with context, assignee, and estimated points.

---

## Factory Sourcing: The Alibaba/Accio Pipeline

This is the workflow that makes other operators' eyes go wide.

**The task:** Source custom woven labels for a new product line. Find suppliers on Accio (Alibaba's sourcing platform), get quotes, compare, recommend.

**What Alice did:**
1. Pulled artwork specs from design files
2. Drafted outreach messages with correct garment industry terminology (woven vs. printed, damask vs. satin weave, centerfold vs. end-fold — learned during the project)
3. Sent messages through browser automation on Accio
4. Set up monitoring cron running every 30 minutes to scan for vendor replies
5. Navigated Accio via browser to read conversation threads
6. Extracted pricing, MOQs, lead times, shipping terms
7. Built comparison matrix with supplier communication quality scoring
8. Recommended winner
9. Cross-referenced delivery timeline against production calendar in Notion

**What went wrong (and what we learned):**
- Sent message to wrong vendor (Mavis instead of Bella) → Rule: verify context before acting on external systems
- Spammed Cordell with monitoring notifications → Rule: monitoring needs rate-limiting and consolidation
- Declined backup vendors after selecting primary → Rule: never close optionality without explicit approval

**Cost comparison:** A sourcing agent or freelancer would cost $500-2,000 for this project. Alice did it for roughly $15 in compute over two weeks.

**Lasting impact:** The garment industry terminology Alice learned during this project persists in her solution files. Future sourcing tasks start from knowledge, not from zero.

---

## Cost Economics: The $3/Day Breakdown

### Before Optimization
Everything running on Claude Opus: $8-12/day (~$240-360/month).

### After Model Routing

| Operation | Model | Cost Per Run |
|-----------|-------|-------------|
| User conversations | Opus | Premium (worth it) |
| Email triage | Haiku | ~$0.02 |
| Background crons (26 jobs) | Haiku | ~$0.001 each |
| Builder sessions | Kimi K2.5 | ~$0.50/hour |
| Vector embeddings | Gemini | Free |
| CX auto-resolve | Python/rules | $0 |

**Daily total: ~$3/day ($90/month)**

### What $90/Month Replaces

| Alternative | Monthly Cost |
|-------------|-------------|
| Human Chief of Staff (full-time) | $6,700-12,500 |
| Fractional Ops Manager | $3,000-5,000 |
| Virtual Assistant | $1,500-3,000 |
| Serif.ai subscription | Variable |
| Linear subscription (team) | $80+ |
| Multiple SaaS tools combined | $200-500 |
| Alice | $90 |

The cost comparison undersells the advantage. Alice has more context than any human hire — she's read every email, every Slack message, every meeting transcript, every Notion task. No human onboards that fast.

---

## Overnight Builder Results

### What Gets Built While Sleeping

The overnight builder system selects tasks using a scoring function (urgency × proximity × staleness × domain rotation) and executes autonomously with budget controls.

**Documented overnight completions:**
- Complete Alice dashboard (Next.js app with token economy, builder progress, cron status)
- ShipBob data warehouse (CSV → SQLite → SQL queries → Google Sheets pipeline)
- Multiple workspace cleanup and documentation tasks
- Code refactoring and technical debt reduction

**Reliability mechanisms:**
- Stall detection: watchdog cron checks builder progress file age, kills hung processes after 20 minutes of inactivity
- Budget cap: never exceeds 60% of remaining daily tokens, 15% per-task maximum
- Delivery guarantee: completion → immediate Telegram notification with screenshots and structured handoff
- Domain rotation: prevents any single domain from monopolizing overnight capacity

### The Delivery Guarantee

Born from three identical failures (February 2, 3, and 9):
1. Spawn builder → "I'll check on it"
2. Session compacts (context lost)
3. Builder completes, no one notified
4. Cordell wakes up disappointed

Hard rule: **Every spawn IMMEDIATELY gets a paired monitoring cron. No exceptions.** The monitoring cron checks for completion and notifies immediately.

---

## Memory System Performance

### Search Coverage
- **QMD index:** 1,457+ files indexed, 224MB SQLite database
- **Hybrid search:** 70% vector similarity + 30% BM25 text matching
- **Embedding model:** embeddinggemma-300M (quantized, runs locally)
- **Cache:** 50,000-entry embedding cache

### Resilience
Four independent paths to memory retrieval:
1. OpenClaw built-in memory search (local embeddings)
2. QMD BM25 text search
3. QMD vector search (via Ollama)
4. Direct file read (fallback of last resort)

**February 15 incident:** Gemini API rate limits killed memory search for an entire day. Built local embedding fallback that same day. Memory search has not failed since.

### Fact Extraction Volume
- 4 extraction runs per day
- ~20 solution files accumulated covering technical, operational, and process learnings
- Weekly synthesis cron consolidates daily notes into long-term memory
- 15+ operating rules promoted from daily notes into AGENTS.md

---

## Compound Learning Metrics

### Rule Accumulation
The system has documented 15+ operating rules, each born from a specific incident:

| Rule | Incident Date | What Happened |
|------|--------------|---------------|
| Spawn → Monitor (Rule 5) | Feb 2, 3, 9 | Three identical spawn-and-forget failures |
| Never run builds in main (6a) | Feb 16 | Cache layer build flooded Telegram |
| Never race on same file (6) | Feb 10 | Agent and subagent wrote conflicting outputs |
| Defer urgent analysis (6b) | Feb 16 | Said "tomorrow" to CSV analysis user needed NOW |
| Verify repo after push (14) | Feb 28 | .gitignore excluded Xcode project file |
| Trial reminders need files (13) | Feb 28 | Reminder fired to non-existent tracking file |
| Never claim "no access" (12) | Feb 28 | Said "no Notion access" when API key existed |
| Exec errors ≠ reply (15) | Feb 28 | Git error ate the Telegram reply slot |

Each rule includes enforcement: it's written into AGENTS.md, which Alice reads at the start of every session.

### Learning Velocity
- **Week 1:** 3 rules from woven labels sourcing
- **Week 2:** 5 rules from builder and delivery failures
- **Week 3:** 4 rules from consolidation audit
- **Week 6:** 3 rules from operational incidents
- **Ongoing:** ~1-2 new rules or refinements per week

The system gets measurably better every week, not through software updates but through institutional knowledge accumulation.

---

## What's Not Working (Honest Assessment)

**Sprint completion rates vary wildly.** Sprint 58 showed 0% completion at 3 PM on the final day. This reflects human execution gaps more than system failure — Alice tracks and reports accurately, but she can't make humans do the work.

**Email draft quality is inconsistent.** Alice's drafts need review 100% of the time. The Outbound Gate exists because fully autonomous email isn't trustworthy yet. Draft quality is good enough to save significant time, but not good enough to send without human eyes.

**Overnight builders occasionally produce low-quality output.** When the model (Kimi K2.5) is tasked beyond its capability, output quality drops. The 3-strike rule exists to catch this: if an agent fails the same task 3 times, stop spawning and escalate.

**Some integrations are fragile.** Himalaya email auth tokens expire and need manual re-authentication. Granola MCP tokens need periodic refresh scripts. These are maintenance taxes on the system.

**[NEEDS CORDELL INPUT]:** Specific numbers on email draft acceptance rate (what % of Alice's drafts get sent with minimal edits vs. significant rewrites). Also: any customer-facing incidents caused by Alice's actions that required apology or correction.

---

## The Compounding Effect

The most important metric isn't any single number. It's the trajectory.

Week 1: Alice was a chatbot that could read email. Week 7: Alice manages email triage, sprint planning, factory sourcing, customer support, overnight builds, and institutional knowledge — with self-healing, budget controls, and compound learning.

Every week adds capabilities. Every incident adds resilience. Every interaction adds context. The system that exists today is strictly better than the system that existed last week, and next week's system will be strictly better than today's.

That's not a feature. It's an architecture. And it's the core argument for treating AI as infrastructure rather than as a tool.

---

*All metrics in this document are from the live Alice system running Superculture Studios operations. No numbers are fabricated or projected. Where data is unavailable, it's marked as [NEEDS CORDELL INPUT].*

---

*← Back to [README](../README) | Section: [The Alice Story](./)*
