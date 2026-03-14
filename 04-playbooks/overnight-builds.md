---
title: "Overnight Builds Playbook"
---

# Overnight Builds Playbook

*How to get 8 hours of autonomous work while you sleep — and not wake up to a mess.*

---

## The Concept

While Cordell sleeps, Alice evaluates pending tasks, selects the highest-value work, spawns specialized agents, monitors their progress, and delivers results by morning. The system has an effective 24-hour workday without burning out any humans.

This isn't speculative. Overnight builders have shipped a complete dashboard, built data warehouse pipelines, and completed documentation and code refactoring tasks.

---

## The Nightshift System

### Task Scoring

Every potential overnight task gets scored:

```
Score = urgency × proximity × staleness × domain_weight
```

| Factor | Description | Example |
|--------|-------------|---------|
| Urgency | How overdue or time-sensitive | Overdue task = high urgency |
| Proximity | Relevance to tomorrow's work | Meeting prep for tomorrow = high |
| Staleness | Days since last attention | Untouched for 5 days = high |
| Domain weight | Rotation bonus for neglected domains | If all recent work was code, ops gets bonus |

**Domain rotation** is key: the system deliberately cycles through different areas (codebase, communications, research, operations, strategic prep) so nothing gets permanently neglected.

### Budget Controls

| Control | Value | Purpose |
|---------|-------|---------|
| Nightly budget cap | 60% of remaining daily tokens | Preserve budget for morning operations |
| Per-task cap | 15% of nightly budget | Prevent one runaway task from eating everything |
| Domain allocation | 30% code, 25% comms, 20% intel, 15% ops, 10% strategic | Balanced coverage |

### Task Selection (1 AM PST)

```
1. Calculate remaining token budget
2. If &lt; 40% of daily budget → skip nightshift (not enough headroom)
3. Score all pending tasks
4. Select top 3-5 across domains
5. Verify each task fits within per-task budget cap
6. Spawn isolated subagents
```

---

## The Builder Pattern

### Spawn Protocol

```
1. Define task clearly in specification
   - What to build (not how)
   - Success criteria
   - File paths for output
   - Constraints (no external communication, branch-only)
   
2. Select agent (Builder for code, Scout for research, etc.)

3. Spawn session with spec

4. IMMEDIATELY create paired monitoring cron (Rule 5 — NON-NEGOTIABLE)

5. Create progress file mechanism:
   echo "$(date +%s)" > /tmp/builder-last-progress-$SESSION_ID
```

### Progress Tracking

Each builder writes a timestamp to its progress file every iteration:

```bash
# Builder writes this each loop
echo "$(date +%s)" > /tmp/builder-last-progress-$SESSION_ID
```

The watchdog cron reads this file periodically to determine builder health.

### Stall Detection

Paired monitoring cron checks progress file age:

```bash
LAST_PROGRESS=$(cat /tmp/builder-last-progress-*)
ELAPSED=$(($(date +%s) - LAST_PROGRESS))

if [ $ELAPSED -gt 1200 ]; then  # 20 minutes
  kill $BUILDER_PID
  notify "Builder stalled after ${ELAPSED}s — killed hung process"
fi
```

**Why 20 minutes?** Too short (5 min) kills builders that are thinking through complex problems. Too long (60 min) wastes hours on genuinely stuck processes. 20 minutes was calibrated through experience.

### macOS Timeout Workaround

Darwin lacks the `timeout` command. Use background PID + kill:

```bash
# Start builder
builder_command &
BUILDER_PID=$!

# Set timeout
(sleep $TIMEOUT_SECONDS && kill $BUILDER_PID 2>/dev/null) &
TIMEOUT_PID=$!

# Wait for builder
wait $BUILDER_PID 2>/dev/null

# Cancel timeout if builder finished first
kill $TIMEOUT_PID 2>/dev/null
```

### Delivery Guarantee

Builder completes → IMMEDIATE delivery. Same turn. No delay.

```
1. Detect completion (progress file + session status)
2. Screenshot results (if visual)
3. Send Telegram notification with:
   - What was built
   - File paths
   - Screenshots
4. Write structured handoff:
   ## HANDOFF
   ### Done: [deliverables with file paths]
   ### Remaining: [next steps]
   ### Decisions Made: [what and why]
   ### Uncertain: [risks, unknowns]
```

**This is a hard rule.** Three identical spawn-and-forget failures (Feb 2, 3, 9) proved that "I'll check on it" doesn't survive session compaction. The monitoring cron + immediate delivery is the only reliable pattern.

---

## The Morning Digest

At 9 AM, Cordell gets one consolidated message:

```
🌙 Overnight Report

✅ Completed:
- Dashboard v3 built and deployed (Builder, 4.2 hours)
  → /workspace/dashboard/ — screenshots attached
- ShipBob data pipeline updated (Builder, 1.8 hours)
  → /workspace/scripts/shipbob-etl.js

⏸ Paused (budget limit):
- Obsidian vault reorganization (ran out of nightly budget at 65%)

❌ Failed:
- Email template redesign (Builder stalled 3x — escalated per 3-strike rule)
  → Needs manual review: model couldn't parse the HTML template

💰 Budget used: $4.20 (52% of nightly allocation)
```

### What's NOT in the Digest
- Raw build logs
- Error stack traces
- Intermediate progress reports

Those stay in daily notes for debugging if needed. The morning digest is executive-level: what got done, what needs attention.

---

## Safety Controls

### Everything Is a Draft

Overnight work follows strict rules:
- **Code:** Written to branches, never merged to main
- **Emails:** Saved to drafts, never sent
- **Documents:** Tagged `[OVERNIGHT REVIEW]`
- **Tasks:** Created as candidates, not approved

Nothing goes live without morning review.

### The 3-Strike Rule

If a builder fails the same task 3 times:
1. STOP spawning
2. Document what's failing (not just "try again harder")
3. Escalate to Cordell in morning digest
4. Do NOT burn more tokens on attempt #4

### Budget Protection

If the nightly budget cap (60%) is reached:
- All remaining builders are paused (not killed — they can resume tomorrow)
- Morning digest reports what was paused and why
- No risk of overrunning into next day's budget

---

## Common Failure Modes

### Model Quality Issues
**Problem:** Kimi K2.5 (free tier) occasionally produces incoherent output, especially on complex tasks.
**Mitigation:** 3-strike rule + canary tasks for unfamiliar work.

### Rate Limit Cascading
**Problem:** Multiple builders spawned simultaneously all hit Kimi K2.5 free tier limits.
**Mitigation:** Fallback chains (Kimi → Sonnet → Opus). Stagger builder start times by 5-10 minutes.

### Progress File Conflicts
**Problem:** Multiple builders writing to `/tmp/builder-last-progress-*` with wildcard matching.
**Mitigation:** Include session ID in filename: `/tmp/builder-last-progress-$SESSION_ID`.

### Stale Sessions
**Problem:** Builder session remains "active" in OpenClaw but the process has died without updating progress.
**Mitigation:** Watchdog checks both progress file age AND session status. Kill if either indicates stall.

---

## Setting Up Your First Overnight Build

### Prerequisites
- Builder agent configured with fallback chain
- Cron system working (verify with a simple test cron)
- Progress file mechanism tested
- Telegram notifications working

### First Night Checklist
1. Choose a simple, well-defined task (e.g., "refactor file X per these rules")
2. Write clear task specification with success criteria
3. Spawn builder at 10 PM (before bed)
4. Create monitoring cron manually (don't trust "I'll check on it")
5. Set morning alarm → check Telegram for results
6. Review: Did it work? Was quality acceptable? Did monitoring catch stalls?

### Scaling Up
After 3+ successful overnight sessions:
- Enable the nightshift cron (1 AM automated task selection)
- Increase from 1 task to 3-5 per night
- Add domain rotation
- Trust the system — but keep the morning review

---

*Overnight builders represent the highest ROI in the system: $4-8 of compute produces work equivalent to $200-500 of freelancer output. Over a month, that's $5,000-10,000 of equivalent value for $120-240 in API costs.*

*See also: [Model Resilience](../tier2/model-resilience.md) for fallback chains, [Multi-Agent Orchestration](../tier2/multi-agent.md) for spawn patterns, [Compound Learning](../tier2/compound-learning.md) for the rules born from builder failures.*

---

*← Back to [README](../README.md) | Section: [Playbooks](./)*
