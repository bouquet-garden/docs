# Multi-Agent Orchestration

*How 9 agents coordinate without stepping on each other — and the failures that taught us how.*

---

## Why Multiple Agents

The naive approach: one AI, one model, one conversation. Simple, but it breaks in three ways.

**Cost.** Running email classification on Claude Opus costs 50x more than Haiku, with identical results for that specific task. One model means paying premium prices for commodity work.

**Context pollution.** When a builder generates 500 lines of code in the main session, that code sits in context for every subsequent interaction. The morning briefing now pays for 500 lines of irrelevant code. Context length inflates, relevance degrades, costs spike.

**Parallelism.** One agent can only do one thing at a time. If Alice is supervising a coding task, she can't simultaneously triage email. Specialization enables parallel execution.

> It's like hiring one person to be your CTO, CMO, and COO. Stop making your AI do everything in one session.

---

## The Agent Roster

### Core Team (Always Available)

| Agent | Model | Role | Memory |
|-------|-------|------|--------|
| **Alice** (main) | Opus | Supervisor, user-facing, strategic orchestration | Full |
| **Builder** | Kimi K2.5 → Sonnet → Opus | Web development, API design | None |
| **Mojo** | Kimi K2.5 → Sonnet → Opus | Mobile/React Native development | None |
| **Phoenix** | Sonnet | Infrastructure, documentation, DevOps | Yes |
| **Scout** | Sonnet | Research, intelligence gathering | Yes |
| **Frank** | Haiku | Operations monitoring, budget watchdog | None |
| **Annie** (CS) | Kimi K2.5 → Sonnet | Email triage, customer support | Yes |

### On-Demand (Spawned for Specific Tasks)

Builder and Mojo agents aren't persistent — they're spawned when coding work is needed, given a task file, and terminated on completion. This keeps overhead near zero when no development work is happening.

### The Decision Autonomy Framework

Not all agents have the same level of authority:

| Level | Agents | Autonomy |
|-------|--------|----------|
| **Supervisor** | Alice | Broad — can make strategic decisions, delegate, approve |
| **Lead** | Builder, Phoenix | Domain autonomy within patterns, Alice approval for cross-domain |
| **Specialist** | Mojo, Scout, Annie | Domain autonomy, escalation-first for anything novel |
| **Intern** | Frank | Narrow autonomy — monitoring, reporting, alerting only |

This framework prevents both paralysis (agent waits for permission on trivial decisions) and overreach (agent makes consequential decisions without approval).

---

## The M\*-TASK.md Pattern

The orchestration breakthrough that made multi-agent coding work.

**Problem:** When you spawn a builder agent, how do you give it enough context to work independently without polluting the main session?

**Solution:** Each task is a self-contained markdown file with everything the agent needs:

```markdown
# M3-TASK.md — Dashboard Widget System

## Context
[What the project is, where it lives, what's been done]

## This Milestone
[Specific deliverables for this agent]

## Technical Constraints
[Stack details, patterns to follow, things to avoid]

## Acceptance Criteria
[How to know it's done]

## Completion Protocol
1. Run build + tests
2. Write HANDOFF section
3. Push to feature branch
4. Signal completion
```

**Why this works:**
- **Agents are stateless.** All context is in the file. No session history needed.
- **Any agent can pick up any task.** If an agent fails, respawn with the same file.
- **Progress survives session boundaries.** The task file + git branch are the state.
- **No coordination overhead.** Each agent works independently on its milestone.

**Proof it works:** The OpenClaw Companion iOS app — 5 agents, 5 milestones (M0 through M4), 4,464 lines of React Native shipped in 41 minutes. Each agent read its M\*-TASK.md file, coded against its feature branch, wrote a HANDOFF section, and Alice merged. Zero inter-agent communication needed.

---

## Spawn Lifecycle

Every agent spawn follows this lifecycle:

```
1. Alice identifies work that needs a specialist
2. Alice writes M*-TASK.md with full context
3. sessions_spawn() creates the agent session
4. IMMEDIATELY: add monitoring cron (Rule 7 — non-negotiable)
5. Agent works from task file
6. Progress tracked via file changes, not session polling
7. Agent writes HANDOFF section on completion
8. Completion auto-announces to Alice
9. Alice reviews deliverable
10. If acceptable → merge to main
11. If not → spawn fresh agent with updated task file
```

### Rule 7: Spawn → Immediately Monitor

This rule exists because of three documented incidents of the same failure:

- **Feb 2:** Builder finished in 22 minutes. Alice told Cordell "30-45 min, I'll check on it." Didn't notify for 3 hours.
- **Feb 3:** Builder completed at 00:06 AM with full reports. Alice never notified. Cordell discovered it 1.5 hours later.
- **Feb 9:** Review-polish agent spawned overnight. Session compacted. Nothing delivered. Cordell woke up disappointed.

Three times. Same pattern. Spawn → forget → disappoint.

Now it's a hard rule: `sessions_spawn()` → IMMEDIATELY `cron.add(monitor)`. No exceptions. The monitor checks completion status and alerts Alice and Cordell when work is done.

---

## Stall Detection

Agents stall. Rate limits hit. Models hallucinate infinite loops. Edge cases cause thrashing.

**Detection:** Every builder writes progress to `/tmp/ralph-last-progress-{hash}`. A watchdog checks file timestamps. No file change for 20 minutes → investigate. No change for 1 hour → kill and assess.

**The Ralph Builder pattern** (see [Unique Tools](unique-tools.md)) enforces timeouts at the loop level. Each coding iteration gets a configurable timeout (default 300 seconds). If Claude Code hangs, the watchdog kills the process and moves to the next loop iteration.

**3-Strike Kill Switch:** If an agent fails the same task 3 times:
1. STOP spawning
2. Write up what's failing (not just "try again harder")
3. Escalate to Cordell with the write-up
4. Do NOT keep burning tokens hoping attempt #4 works

This rule prevents the "hope-based engineering" anti-pattern where you keep respawning a failed agent without understanding why it's failing.

---

## Rule 6: Never Race a Subagent on the Same File

**The incident:** Alice deep-dived the Campsite codebase (read 15+ files, analyzed patterns), then spawned a Sonnet subagent to "write the spec" from a prompt summary. The subagent produced 2,006 lines with **fabricated code examples** — generic shadcn CSS instead of actual Campsite code. Meanwhile, Alice wrote 1,306 lines with real code from the deep-dive. Both targeted the same file. Alice's version won by luck (committed 2 minutes later).

**The rules this spawned:**
1. If you have the context, YOU write it. Subagents work from prompts, not your memory.
2. Never have two writers targeting the same file path.
3. If a subagent looks stalled, check the OUTPUT FILE (`ls -la`), not just the session.
4. Quality degrades when subagents work from descriptions of code instead of actual code.

---

## Rule 6a: Never Build in Main Session

The main Telegram session is Cordell's interface, not a build terminal.

**The incident (Feb 16):** Alice built an entire Granola cache layer directly in the main session — cache manager, CLI upgrade, warming 21 meetings, fetching 7 transcripts, testing cache-hit flow. Every curl, every cache-miss, every stat check showed up as Telegram noise.

**The rule:** If it involves multiple API calls, cache warming, file generation loops, bulk fetching, script testing with retries, or anything that takes >2 tool calls to complete → spawn a subagent.

**The test:** Before running a tool, ask: "Would Cordell want to see this in Telegram?" If no → sub-agent.

---

## Structured Handoffs (Rule 10)

Every subagent completion must end with a structured handoff — not a prose summary.

```markdown
## HANDOFF
### Done: [specific deliverables with file paths]
### Remaining: [specific next steps with enough context for a fresh agent]
### Decisions Made: [what was decided and WHY — prevents re-litigation]
### Uncertain: [risks, unknowns, things to investigate]
```

**Why:** Prose summaries ("I completed all 5 priorities!") lose critical context across sessions. A fresh agent — or Alice — picks up from exact state, not vibes.

**The incident (Feb 12):** An overnight builder dumped a celebratory summary with no remaining/uncertain items. The next session had to re-read all the code to figure out actual state. Structured handoffs prevent this.

---

## Fresh Eyes Review Loop (Rule 9)

After completing any significant deliverable, spawn 2 independent review agents:

1. Each reviewer gets the deliverable + brief context (NOT the session history)
2. Reviewers independently assess using P1/P2/P3 classification
3. Fix all P1 and P2 findings
4. Re-run reviews on the fixed version
5. Repeat until clean (max 3 rounds)

**Why fresh eyes:** You can't proofread your own work. Confirmation bias affects agents just like humans. A reviewer with no session history sees the deliverable as a new reader would.

**The model:** Use Sonnet for reviews — good balance of quality and cost. Two reviewers catch different things: one finds logic gaps, the other finds UX issues.

**Constraint:** Max 3 iterations. If still finding P1s after 3 rounds, something is structurally wrong — escalate to Cordell.

---

## The Coordination Reduction

The original plan had 8 agents (Alice, Dexter, Mojo, Phoenix, Scout, Cortex, Comms, CS). The framework critique identified n² coordination complexity — 28 possible agent pairs.

**The consolidation:** Merged Cortex → Phoenix, Comms → Alice. Reduced core team to 4 active + 3 on-demand. Coordination pairs dropped from 28 to manageable.

**The insight from Amazon's Two-Pizza Teams principle:** If a team is too big to feed with two pizzas, it's too big to coordinate. The same applies to agent teams.

---

*← Back to [README](../README.md) | Related: [Unique Tools](unique-tools.md) · [Agent System](../01-foundations/agents.md)*
