---
title: "Sprint Management Playbook"
---

# Sprint Management Playbook

*How Alice manages sprints in Notion — and why we replaced Linear.*

---

## Why Notion Over Linear

Linear is a great product. We chose Notion instead because Alice can keep Notion honest in ways no standalone tool can.

The differentiator isn't the tool — it's the cross-reference intelligence. Alice reconciles what's in Notion against what's actually happening in email, Slack, and meeting transcripts:

- "We discussed this in Slack but it's not in Notion" → surface it
- "This email implies the task is done but Notion says In Progress" → flag it
- "New email from client implies urgent work not captured in sprint" → create task candidate

No standalone project management tool can do this because no standalone tool has access to all your communication surfaces.

---

## Sprint Cycle

### Weekly Setup (Monday Morning)

1. **Review previous sprint scorecard** — What completed? What rolled over?
2. **Pull Notion backlog** — Unstarted tasks, deferred items, new requests
3. **Enrich from communication** — Check email and Slack for work not captured in Notion
4. **Assign priorities** — P0 (must complete), P1 (important), P2 (nice to have)
5. **Estimate points** — Based on similar past tasks
6. **Assign owners** — Cordell, team members, or Alice
7. **Set sprint in Notion** — Tasks tagged with sprint number

**Target: 35-40 points per sprint** (calibrated from Sprint 55 baseline at 39 points / 94% completion).

### Daily Operations

**Morning:** Sprint status in the morning digest. Any overdue items highlighted.

**Throughout day:** Task candidates surface from email/Slack via inline Telegram buttons:
```
📬 New task candidate from email:
"Schedule call with Kevin about pricing"
[✅ Approve] [⏭ Skip] [✏️ Edit]
```

**EOD:** Task movement summary. What progressed, what's stuck.

### Friday Scorecard

Automated sprint scorecard with:

| Metric | Format |
|--------|--------|
| Completion rate | X/Y tasks (Z%) vs target |
| P0 hit rate | X/Y P0 tasks complete |
| Total points | Actual vs planned |
| Per-person breakdown | Tasks, points, completion % per team member |
| Analysis | What went well, what went wrong, recommendations |

**Example from Sprint 58:**
```
Completion: 0/26 (0%) — Target: 94%
P0 Hit Rate: 0/3 (0%) — Target: 100%
Total Points: 48 — Target: 39

Analysis: Sprint overloaded at 48pts vs 39pt baseline.
P0 tasks deprioritized. One P0 task blocked on dependencies.
```

The scorecard doesn't pull punches. It reports what happened, not what anyone wanted to happen.

---

## Task Inference Engine

Emails and Slack messages often contain implicit tasks that never make it into Notion:

```
Email signal detection
    ↓ (pattern matching: deadlines, requests, invoices)
Confidence scoring (0-10)
    ↓ (threshold: 5+ → create candidate)
Deduplication vs Notion
    ↓ (fuzzy match on subject/content)
Task candidates (task-candidates.json)
    ↓ (human review via Telegram buttons)
Notion task creation (with context)
```

**Signals detected:**
- Deadline language ("by Friday," "end of week")
- Request patterns ("can you," "please send," "we need")
- Invoice/payment mentions
- Follow-up indicators ("following up on," "checking in on")

**Deduplication:** Before creating a candidate, Alice checks existing Notion tasks for fuzzy matches. If "Schedule call with Kevin" already exists, no duplicate is created.

---

## Cross-Reference Validation

The highest-value sprint management capability:

### Email → Notion
Every email triage run checks: does this email relate to an existing sprint task? If a vendor responds to a production order, and there's a Notion task for that order, Alice connects them.

### Slack → Notion
Activity digest identifies Slack threads that mention projects or tasks. If a discussion implies work that isn't tracked, it gets surfaced.

### Meeting Transcripts → Notion
When a Granola transcript is processed, action items are extracted and matched against existing tasks. New action items become task candidates. Existing tasks get updated with meeting context.

---

## Sprint Health Indicators

| Indicator | Green | Yellow | Red |
|-----------|-------|--------|-----|
| Completion rate | >80% | 60-80% | &lt;60% |
| P0 hit rate | 100% | 1 missed | >1 missed |
| Points vs capacity | Within 10% | 10-25% over | >25% over |
| Blocked tasks | 0 | 1 | >1 |
| Untracked work | &lt;10% | 10-25% | >25% |

**Untracked work** is the sneaky one: work happening in email and Slack that never gets captured in Notion. The cross-reference validation catches this, but it requires the world state cache to be current.

---

## Playbook: Setting Up Sprint Management

### Prerequisites
- Notion API key configured (`~/.config/notion/api_key`)
- Notion database with: Task name, Status, Priority, Points, Assignee, Sprint, Due date
- World state cache active (cron refreshing Notion, email, Slack data)

### Initial Setup
1. Create sprint database in Notion with required fields
2. Configure Notion API integration in OpenClaw
3. Set up morning digest cron to include sprint status
4. Set up task candidate pipeline (email signal detection → Telegram buttons)
5. Set up weekly scorecard cron (Friday 3 PM)

### Tuning
- **Sprint capacity:** Start at 30 points, adjust based on completion rate over 3+ sprints
- **Point estimation:** Compare estimates vs actuals to calibrate
- **P0 allocation:** Max 3 P0 tasks per sprint (more than 3 = everything is urgent = nothing is)
- **Cross-reference aggressiveness:** Too many false connections create noise; tune matching thresholds

---

## Lessons Learned

1. **Sprint overload is the #1 failure mode.** Sprint 58 had 48 points vs 39 baseline → 0% completion. Better to under-commit and add than over-commit and miss.

2. **P0 tasks must be prioritized first.** If P0s aren't done by Wednesday, escalate. Waiting until Friday guarantees misses.

3. **Cross-reference validation catches real gaps.** Email conversations that should be tasks, Slack decisions that should be documented — the system catches what humans forget.

4. **The scorecard is accountability.** It's uncomfortable when it shows 0% completion. That discomfort drives behavior change.

5. **[NEEDS CORDELL INPUT]:** Ideal sprint structure and cadence. Are weekly sprints right, or should they be bi-weekly? What's the target completion rate after calibration?

---

*See also: [Operational Results](../tier3/operational-results.md) for sprint metrics, [Workflows & Crons](../tier1/workflows.md) for the cron setup.*

---

*← Back to [README](../README.md) | Section: [Playbooks](./)*
