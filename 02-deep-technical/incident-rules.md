---
title: "Incident-Derived Rules: The System's Immune System"
---

# Incident-Derived Rules: The System's Immune System

*18 rules. Each one born from a real failure. Each one preventing an entire class of errors permanently.*

---

## Why This Matters

Every rule in Alice's operating system (AGENTS.md) has a story behind it. Not a hypothetical scenario — a real failure that cost real time, disappointed a real person, or broke a real workflow.

This is what separates a production AI system from a demo. Demos don't have scar tissue. Production systems do.

> "Rule 12 exists because Alice told me she couldn't access Notion when the API key was sitting right there in the config directory."

---

## The Rules

### Rule 6: Never Race a Subagent on the Same File

**The Incident (Feb 10):** Alice deep-dived the Campsite codebase — read 15+ files, analyzed patterns, understood the architecture. Then she spawned a Sonnet subagent to "write the spec" from a prompt summary.

The subagent produced 2,006 lines with **fabricated code examples** — generic shadcn CSS instead of actual Campsite code. It was working from a description of the codebase, not the codebase itself.

Meanwhile, Alice wrote 1,306 lines with real code from her deep-dive. Both wrote to the same file. Alice's version won by luck — committed 2 minutes later.

**The Rule:**
- If you have the context, YOU write it. Subagents work from prompts, not your memory.
- Never have two writers targeting the same file path.
- Quality degrades when subagents work from descriptions instead of actual code.

---

### Rule 6a: Never Build in Main Session

**The Incident (Feb 16):** Built the entire Granola cache layer directly in the main Telegram session. Cache manager, CLI upgrade, warming 21 meetings, fetching 7 transcripts, testing cache-hit flow. Every curl, every cache-miss, every stat check showed up as Telegram noise. Cordell's phone buzzed with raw HTTP responses.

**The Rule:** If it takes >2 tool calls to complete → spawn a subagent. The main session is Cordell's interface, not a build terminal.

**The Test:** "Would Cordell want to see this in Telegram?" If no → sub-agent.

---

### Rule 6b: Never Defer Urgent Analysis

**The Incident (Feb 16):** Cordell sent an 875-contact CSV for warm lead analysis during an active work session. Instead of spawning a subagent to parse it immediately, Alice saved the file and said "Tomorrow: I analyze the contacts." Cordell was in execution mode — "tomorrow" was unacceptable. 30 minutes wasted.

**The Rule:**
- If user sends data during active session → start analysis within 60 seconds
- Never say "tomorrow" for deliverables the user needs NOW
- Run parallel work tracks when tasks are independent

---

### Rule 7: Spawn → Immediately Monitor (CONTEXT.md Compaction Survival)

**The Incidents (Feb 2, Feb 3, Feb 9):** Three documented instances of the exact same failure:

1. **Feb 2:** Builder finished in 22 minutes. Alice told Cordell "30-45 min, I'll check on it." Didn't notify for 3 hours.
2. **Feb 3:** Builder completed at 00:06 AM with full reports. Alice never notified. Cordell discovered it 1.5 hours later.
3. **Feb 9:** Review-polish agent spawned overnight. Session compacted. Nothing delivered. Cordell woke up disappointed.

Three times. Same pattern. Spawn → say "I'll check" → session compacts → work disappears.

**The Rule:** `sessions_spawn()` → IMMEDIATELY `cron.add(monitor)`. No exceptions. No "I'll check on it." The monitor is the check.

**Related:** CONTEXT.md was created specifically to survive compaction. A compact (&lt;100 lines) file capturing current operational state, updated after every significant action. When compaction wipes session context, CONTEXT.md tells future-you where things stand.

---

### Rule 8: Build Orchestration Guardrails

**The Incidents:**
- **Config patch array replacement (Feb 11):** When patching `agents.list`, included only the 2 agents being changed. The patch replaced the entire array — 5 agents were accidentally deleted from config.
- **Model ID bulk migration:** 19 of 28 crons migrated from GLM-free to Haiku using wrong format (`claude-haiku-3-5` instead of `claude-3-5-haiku`). 5 actively failing.

**The Rules:**
- Config patches with arrays MUST include ALL items (the patch replaces, doesn't merge)
- After bulk operations, validate at least one runs before assuming all work
- Canary task before real work on unfamiliar model/task combinations
- Git branch per agent (`{agent-name}/{milestone}-{feature}`)
- 3-strike kill switch: 3 failures on same task → stop, write up, escalate

---

### Rule 9: Fresh Eyes Review Loop

**The Problem:** Alice can't proofread her own work. Confirmation bias applies to AI just like humans. The same session that produced the work will rubber-stamp the review.

**The Rule:** After completing any significant deliverable:
1. Spawn 2 review subagents with fresh context (no prior involvement)
2. Each reviews independently using P1/P2/P3 classification
3. Fix all P1 and P2 findings
4. Re-review with fresh sessions
5. Repeat until clean (max 3 rounds)
6. If P1s persist after 3 rounds → escalate to Cordell

**In practice:** The IP licensing case study went through 2 rounds of review (4 P1s each round, all fixed cleanly). Without fresh eyes, those P1s would have shipped.

---

### Rule 10: Structured Handoffs

**The Incident (Feb 12):** An overnight builder dumped a celebratory summary: "All 5 priorities completed! 🎉" No file paths. No remaining work. No decisions made. No uncertainties flagged. The next session had to re-read all the code to figure out actual state.

**The Rule:** Every completion MUST end with:
```
## HANDOFF
### Done: [specific deliverables with file paths]
### Remaining: [specific next steps with enough context for a fresh agent]
### Decisions Made: [what was decided and WHY]
### Uncertain: [risks, unknowns, things to investigate]
```

Prose summaries are not handoffs.

---

### Rule 11: Nightshift Principles

**Born from:** The accumulated wisdom of overnight autonomous operations.

**The Rules:**
- Budget cap: never exceed 60% of remaining daily tokens
- Everything is a draft — emails in drafts, code in branches, docs tagged `[REVIEW]`
- Task scoring: urgency × proximity × staleness × domain rotation
- Morning digest: one consolidated message at wake-up
- Per-task cap: max 15% of nightly budget

See [The Nightshift](nightshift.md) for the full design.

---

### Rule 12: Never Claim "No Access" Without Checking

**The Incident (Feb 28):** Alice told Cordell "I don't have direct Notion API integration." The API key existed at `~/.config/notion/api_key`. Four scripts used it daily. Pure laziness — guessed instead of checking.

**The Rule:**
1. Before saying "I can't access X" → run `which X`, `ls ~/.config/X`, or `grep -r X scripts/`
2. Check the Integration Registry in TOOLS.md
3. If genuinely not available after checking → say "I checked and couldn't find credentials for X"
4. Guessing "no" is worse than spending 10 seconds verifying

> This is one of the most-cited rules in the system. It captures a failure mode that's embarrassing in its simplicity — the AI was too lazy to look.

---

### Rule 13: Trial Reminders Must Create Their Tracking File

**The Incident (Feb 28):** Created a "shaping skills trial review" reminder pointing to `memory/experiments/shaping-skills-trial.md` but never created the file. Reminder fired 2 weeks later to an empty path. The trial happened but was never tracked.

**The Rule:** When setting a future reminder:
1. Create the tracking file IMMEDIATELY — don't defer it
2. The file must exist BEFORE the cron is created
3. Include: what's being trialed, success criteria, usage log section

---

### Rule 14: Verify Repo Contents After Push

**The Incident (Feb 28):** Pushed async-swift to GitHub. `.gitignore` excluded `*.xcodeproj/`. Cordell cloned on his work laptop — no project file, can't build. Wasted his time debugging a missing file.

**The Rule:** After `gh repo create --push` or first push:
1. Run `git ls-files | grep <critical-file>` to confirm key files landed
2. Check `.gitignore` for overly broad rules
3. If a build file is missing from tracked files — fix `.gitignore` and force-add

---

### Rule 15: Never Let Exec Errors Eat the Reply Slot

**The Incident (Feb 28):** Merge conflict during async-swift push leaked raw git errors to Telegram. The actual deliverable message ("all milestones merged") was never sent. Cordell saw git noise, missed the deliverable.

**The Rule:**
1. Send deliverables via `message` tool FIRST — before running exec commands
2. Never let exec errors be your final reply
3. Always follow up with a clean status message regardless of outcome

**The test:** "If this exec fails, will Cordell know what I accomplished?" If no → send status first.

---

## Additional Operational Rules

### "Review" Means Wait
**Incident (Feb 3):** Cordell said "review the spec." Alice wrote the spec then immediately started building without waiting for approval.

### Verify Context Before External Actions
**Incident (Jan 27):** Sent a message to the wrong vendor (Mavis instead of Bella) because chat selection wasn't verified.

### Never Close Optionality Without Approval
**Incident (Jan 28):** After selecting a primary supplier, Alice proactively declined all backup vendors. When the primary had a production delay, re-engaging backups cold cost a week.

### Monitoring Needs Rate-Limiting
**Incident (Jan 27):** Created an email monitoring daemon that spammed Cordell with notifications every few minutes. Three overlapping monitoring systems running simultaneously.

---

## The Pattern

Every rule follows the same lifecycle:

```
Real failure → Documented incident → Root cause analysis 
    → Specific rule → Promoted to AGENTS.md → Read every session 
    → Failure category eliminated
```

After 10 weeks: 18+ rules, each preventing an entire class of errors. The error surface is monotonically shrinking. New failure modes emerge, but old ones are gone forever.

> This is what compound learning looks like in practice. Not fine-tuning. Not prompt engineering. Institutional memory encoded as enforceable rules, born from real scar tissue.

---

*← Back to [README](../README.md) | Related: [Compound Learning](compound-learning.md) · [The Alice Story](../03-the-alice-story/the-story.md)*
