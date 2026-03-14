---
title: "Workflows & Crons"
---

# Workflows & Crons

*26 automated workflows that anticipate needs instead of waiting for instructions.*

---

## The Cron Philosophy

Most people use AI reactively: ask a question, get an answer. Alice's cron system inverts this — the AI monitors, analyzes, and acts proactively across 6 operational domains, 24 hours a day.

The shift from reactive to proactive is the single biggest quality-of-life improvement in the system. Cordell wakes up to a morning briefing. Email gets triaged before he checks his inbox. Stale threads get flagged before they become problems. The system anticipates needs instead of waiting for instructions.

---

## Workflow Catalog

### Email Workflows

**Email Fast Poll** — Checks for urgent emails every 30 minutes during business hours. If something P0 (client crisis, payment issue, team blocker) arrives, Telegram alert fires immediately rather than waiting for the next full triage.

**Morning/Evening Triage** — Full email classification twice daily. Each email categorized (P0-P3), thread ownership determined ("ball with us" or "ball with them"), staleness calculated, drafts prepared for urgent items.

**Follow-Up Tracker** — Identifies stale threads daily. Our ball > 24 hours = warning. Their ball > 48 hours = suggest a follow-up nudge. Prevents balls from being dropped.

**Email Fast Poll (Vendor Hours)** — Monitors specifically during Chinese business hours (4-5 PM PST) for factory and supplier responses. Time-zone aware: catches vendor replies when they're most likely.

### Notion / Sprint Workflows

**Sprint Setup** — Weekly sprint creation. Pulls tasks from Notion backlog, enriches with context from email and Slack, assigns priorities and estimates.

**Sprint Audit** — Daily check against sprint plan. Identifies overdue tasks, missing estimates, blocked items. Cross-references Notion tasks against actual activity in email and Slack.

**Task Digest** — End-of-day summary of task movement. What got done, what moved, what's stuck.

### Slack Workflows

**Reaction Engine** — Monitors Slack channels for emoji reactions:
- 🎫 on a message → creates Notion task with message content as context
- ✉️ → drafts email related to the message
- ⏰ → sets reminder

**Activity Digest** — Daily summary of Slack channel activity with cross-reference to Notion tasks. "This Slack thread relates to Notion task X" connections.

### Knowledge Workflows

**QMD Embedding Cron** (nightly) — Re-indexes new and modified files into the 224MB QMD search database. Keeps semantic search fresh across the Obsidian vault, workspace, and session history.

**Fact Extraction** (4x/day) — Pulls structured facts from conversations. Decisions → daily notes. Lessons → daily notes. Relationship intelligence → daily notes. The raw material for long-term memory.

**Weekly Synthesis** — Reviews the week's daily notes. Promotes persistent learnings to MEMORY.md. Flags rules for AGENTS.md promotion. Cleans up ephemeral entries.

### Monitoring Workflows

**Budget Watchdog** — Frank (Haiku) monitors daily token spend. Computes cascade mode (Normal/Conservative/Austerity/Emergency). Alerts Cordell if budget drops below thresholds.

**Vendor Reply Monitor** — Tracks responses from active suppliers. Time-zone aware scheduling.

**CX Monitor** — Checks Commslayer/Chatwoot for new customer support tickets.

### Hygiene Workflows

**Obsidian Vault Sync** — Git pull every hour. Keeps the local knowledge base in sync with the Obsidian sync service.

**Overnight Builder Check** — Monitors active builder sessions. Checks progress files, detects stalls, ensures delivery on completion.

---

## Scheduling Architecture

### Time-Aware Scheduling

```
6 AM  — Overnight builder results check
9 AM  — Morning digest + standup (consolidated)
10 AM — Email triage (first full run)
11:30 AM — Cross-signal detection (Frank)
12 PM — Fact extraction
2 PM  — Email fast poll
4 PM  — Vendor reply monitor (China 8 AM)
5:30 PM — Cross-signal detection (Frank)
6 PM  — EOD review + task digest
8 PM  — Email triage (second full run)
1 AM  — Nightshift task scoring + builder spawning
4 AM  — Overnight checks
```

### Staggered Minute Marks

Jobs spread across minute marks: 0, 2, 15, 20, 25, 35, 40, 45, 50.

**Why?** Running 5 crons at minute :00 guarantees at least one hits an API rate limit. Staggering by even 2-5 minutes prevents cascading failures.

### Quiet Hours

10 PM - 8 AM: no Telegram notifications except urgent alerts. The system works overnight but doesn't wake anyone up unless something is actually critical.

---

## Key Workflow: Email Triage Pipeline

The most complex and most valuable workflow:

```
1. Pull unread emails via himalaya
   ↓
2. For each email:
   a. Read content and context
   b. Check world state cache for related Notion tasks, Slack threads
   c. Classify priority (P0-P3)
   d. Determine thread ownership ("ball with us" / "ball with them")
   e. Calculate staleness (hours since last activity)
   ↓
3. For P0/P1 emails:
   a. Draft response with Alice notes (red HTML context blocks)
   b. Include: Notion task status, Slack thread references, meeting transcript context
   c. Save to Gmail Drafts (NEVER auto-send)
   ↓
4. Update Gmail labels (1: Needs Response, 3: Waiting for Reply)
   ↓
5. Send Telegram notification: "X urgent drafts ready for review"
   ↓
6. Await explicit "send" instruction before delivery
```

**Safety gates at every step:** Reading is always safe. Drafts saved to folder (safe). Labels applied (safe). Only sending requires explicit approval.

---

## Key Workflow: Overnight Builder

```
1. Nightshift cron (1 AM) evaluates task catalog
   ↓
2. Score tasks: urgency × proximity × staleness × domain rotation
   ↓
3. Budget check: remaining tokens > 40% of daily budget?
   ↓
4. Select top 3-5 tasks across domains (code, comms, research, ops, strategic)
   ↓
5. Per-task budget cap: max 15% of nightly budget
   ↓
6. Spawn isolated subagents for each task
   ↓
7. Create paired monitoring cron for each spawn (Rule 5)
   ↓
8. Monitor: progress file age check every 20 min
   ↓
9. Stall detected? → Kill hung process, alert
   ↓
10. Completion detected? → Immediate Telegram notification + screenshot + structured handoff
    ↓
11. Morning digest (9 AM): consolidated summary of overnight work
```

---

## Key Workflow: Meeting → Actions

```
1. Meeting occurs (captured by Granola)
   ↓
2. Refresh Granola token: bash ~/.config/granola/refresh-token.sh
   ↓
3. Pull transcript via Granola MCP API (50-70K characters)
   ↓
4. Process against business context:
   - Current sprint board (Notion)
   - Outstanding tasks
   - Recent decisions
   - Active projects
   ↓
5. Extract structured outputs:
   - Homework items → Notion tasks (with linked transcript quotes)
   - Mindset shifts → Reference notes
   - Outreach targets → Tasks with suggested messaging
   - Operational changes → Calendar event suggestions
   ↓
6. Deduplicate against existing Notion tasks
   ↓
7. Send Telegram summary with top takeaways + Notion links
```

**What makes this different from meeting summary tools:** Alice knows what Cordell is already working on. She doesn't create duplicate tasks. She understands the delta between what was discussed and what's already in motion.

---

## Cron Model Assignment

All crons run on Haiku ($0.001/run). This is a hard rule born from the GLM-4.7-free incident where ~50% of overnight crons silently failed on the free model.

**The economics:** 26 crons × ~$0.001/run × 2-3 runs/day = $0.05-0.08/day total cron cost. This is negligible. The reliability matters infinitely more than the cost.

**Lesson:** The cheapest model that actually works is infinitely cheaper than one that silently fails.

---

## Disabled Crons (Lessons in Pruning)

5 crons were disabled during the v2.0 audit because they produced no value:

1. Task Digest reading empty JSON files
2. Supervision Check for a pipeline that was never built
3. Context Enrichment with vague prompt generating noise
4. Overnight check for a file that was deleted on day one
5. Duplicate morning notification (merged into single digest)

**Pruning discipline:** Crons compound. Even at $0.001/run, noise from no-op crons obscures real signals. Audit ruthlessly.

---

*26 active workflows, 5 disabled (kept for documentation). Total cron cost: ~$0.08/day. Value delivered: proactive email triage, sprint management, knowledge indexing, budget protection, vendor monitoring, and overnight building — all without being asked.*

*See also: [Infrastructure & Monitoring](../tier2/infra-monitoring) for cron health monitoring, [Email Triage Pipeline](../tier4/email-triage) for step-by-step playbook, [Overnight Builds](../tier4/overnight-builds) for builder patterns.*

---

*← Back to [README](../README) | Next: [Deep Technical](../02-deep-technical/)*
