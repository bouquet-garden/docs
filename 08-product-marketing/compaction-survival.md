# Compaction Survival

**How to give an AI agent persistent memory in a world that keeps erasing it.**

LLMs have a fundamental constraint that most people don't think about until it breaks everything: **finite context windows.** Your conversation gets long enough, the old stuff gets summarized or dropped. The model "forgets."

In a chatbot, this is a minor annoyance. You re-explain what you were talking about.

In an autonomous agent that runs 24/7, manages vendor relationships, processes meetings, and operates overnight — it's an existential threat. If the agent forgets what it was doing mid-task, things break. Tasks get lost. Promises get abandoned. Work gets repeated.

We solved this. The solution is embarrassingly simple, genuinely useful, and — as far as we can tell — an original contribution to the "how do you run persistent AI agents" problem.

It's called `CONTEXT.md`.

---

## The Problem: What Compaction Actually Does

### How LLM context works
Every LLM has a context window — the maximum amount of text it can process at once. Claude's is large (200K tokens). But even 200K tokens fills up during a long work session.

When the context fills, the system must make room. The process is called **compaction**: the LLM summarizes the older parts of the conversation, keeping the recent parts intact. A 60K-token conversation might get compacted to 19K tokens.

### What survives compaction
- Recent messages (last few exchanges)
- System prompts (the agent's instructions)
- A summary of older context (compressed, lossy)

### What doesn't survive compaction
- **Specific task details** from hours ago
- **Promises made** earlier in the session ("I'll check on that builder")
- **Working state** (which files were modified, what's still pending)
- **Decision context** (why a specific approach was chosen)
- **Intent** ("I was planning to do X after Y finishes")

### Why this breaks agents
A chatbot can re-read the conversation summary and pick up the thread. An agent can't, because agents have **obligations** — tasks in progress, promises to fulfill, monitoring responsibilities. When compaction erases the context behind those obligations, the agent doesn't know it has them.

**This happened to us three times** (Feb 2, Feb 3, Feb 9):
1. Alice spawns a sub-agent for a task
2. Alice tells Cordell "I'll check on it"
3. Hours pass. Context fills with other work.
4. Compaction fires. The "I'll check on it" intent is summarized to nothing.
5. The sub-agent finishes. Nobody tells Cordell.
6. Cordell wakes up or asks hours later. Disappointment.

Same failure pattern, three times, before we built the fix.

> *Tweetable: "LLMs forget things. If your AI agent runs for more than a few hours, it WILL forget what it was doing. The question is what you do about it."*

---

## The Solution: CONTEXT.md

`CONTEXT.md` is a compact file — under 100 lines — that captures the agent's **current operational state.** It lives in the workspace root. The agent reads it at the start of every session. The agent updates it after every significant action.

### The key insight
Compaction summarizes *conversation*. It doesn't touch *files.* A file on the filesystem survives compaction perfectly. The agent's working state, externalized to a file, is compaction-proof.

### The format

```markdown
# CONTEXT.md

## Current Project: [name]
## Current Milestone: [M0/M1/etc] - [status emoji] [one-line status]

## Active Agents
| Agent | Task | Branch | Status | Last Update |
|-------|------|--------|--------|-------------|

## Last 3 Decisions (and WHY)
1. [decision] - because [reason]
2. [decision] - because [reason]
3. [decision] - because [reason]

## Don't Re-Litigate
- [settled decision that future-you might question]

## Blockers
- [blocking item]

## Next Action
- [what to do next]
```

### Why it works
When a fresh session starts — or when compaction has just wiped the conversation context — the agent reads CONTEXT.md and immediately knows:

1. **What's happening right now** (current project, milestone, status)
2. **Who's doing what** (active agents and their tasks)
3. **What was recently decided** (and why — preventing re-litigation)
4. **What not to re-debate** (settled decisions)
5. **What's blocked** (so the agent doesn't waste time on blocked paths)
6. **What to do next** (immediate action, no analysis paralysis)

All in under 100 lines. Readable in 5 seconds. The entire operational state of a 4-agent AI team, compaction-proof.

> *Tweetable: "CONTEXT.md: under 100 lines that tell your AI agent everything it needs to know after it forgets everything. The simplest solution to the hardest problem in persistent AI."*

---

## The Rules Around CONTEXT.md

### Rule 7: Update CONTEXT.md before daily notes
CONTEXT.md is more important than daily notes. If you only have time to update one thing, update CONTEXT.md. The daily notes are a detailed log; CONTEXT.md is the survival briefing.

### Max 100 lines
Ruthlessly prune. If CONTEXT.md grows beyond 100 lines, it's trying to be a daily log instead of a state snapshot. Cut completed projects, resolved blockers, old decisions.

### Update after every significant action
Not every action — significant ones. Spawned a sub-agent? Update CONTEXT.md. Made an architectural decision? Update. Completed a milestone? Update. Changed priorities? Update.

The discipline is: before you move to the next thing, externalize the state of the thing you just finished.

### The "Don't Re-Litigate" section
This is the secret weapon. After compaction, the agent might look at a decision and think "why are we using Notion instead of Linear?" Without the Don't Re-Litigate section, it re-researches, re-evaluates, and possibly re-decides — wasting time and potentially introducing inconsistency.

With the section, the agent sees: "Notion over Linear — already evaluated, chose Notion for flexibility and human familiarity (see ADR-006)." Discussion over. Move on.

---

## The Broader Pattern: Externalized State

CONTEXT.md is the most visible implementation of a broader principle: **agents must externalize their working state to files, not rely on context memory.**

### Per-agent WORKING.md
Each agent in the team has a WORKING.md file tailored to their domain:
- **Dexter (code):** Current branch, files modified, tests passing, build status
- **Mojo (design):** Current design refs, component status, screenshot links
- **Phoenix (infra):** CI status, deployment state, environment health
- **Annie (comms):** Active email threads by ID, pending responses, escalations
- **Frank (analytics):** Current analysis state, data sources loaded, findings

When any agent's session gets compacted, they re-read their WORKING.md and know exactly where they left off.

### The M*-TASK.md pattern
For multi-agent builds (like the 41-minute iOS app), each milestone gets a task file:
- `M0-TASK.md`: Calibration task with acceptance criteria
- `M1-TASK.md`: First milestone with specs, context, and progress
- `M2-TASK.md`: Second milestone, etc.

The task file is the complete context. The agent is stateless — it reads the task file, does the work, updates the task file. Compaction can't break this because the state was never in memory; it was always in the file.

### Daily notes: memory/YYYY-MM-DD.md
The detailed log. Everything that happened, decisions made, lessons learned. Not compaction-proof (too long to re-read every session), but serves as the archive that CONTEXT.md is distilled from.

### Long-term memory: MEMORY.md
Curated memories. Not raw logs — distilled insights, preferences, relationship context, lessons learned. Like a human's long-term memory: you don't remember every Tuesday, but you remember the important patterns across Tuesdays.

---

## The Compaction Resilience Checklist

Before any operation expected to take more than 30 minutes:

1. ✅ **Write a checkpoint** to `memory/YYYY-MM-DD.md` with current state
2. ✅ **Update CONTEXT.md** with current project, milestone, and next action
3. ✅ **Ensure sub-agent tasks write progress to files**, not just session context
4. ✅ **Use M*-TASK.md pattern** for builds (task file = full context)
5. ✅ **Check session status proactively** during long sessions

### The anti-pattern
```
❌ "I'll remember to check on the builder later"
❌ "I have all the context in my current session"
❌ "I'll write the summary when I'm done"
```

### The correct pattern
```
✅ "I'm spawning a builder. CONTEXT.md updated. Monitoring cron created."
✅ "Long session ahead. Checkpoint written to daily notes."
✅ "Decision made: using Notion for sprints. Added to Don't Re-Litigate."
```

> *Tweetable: "The biggest risk in persistent AI isn't hallucination. It's amnesia. Every promise your agent can't remember is a promise it won't keep."*

---

## Why This Matters Beyond Alice

### The industry problem
Everyone building AI agents hits this wall. Long-running sessions get compacted. Multi-step tasks lose context. Agents "forget" what they were doing.

The common solutions:
- **Bigger context windows:** Helps, but eventually fills up. Not a solution; a delay.
- **RAG (retrieval augmented generation):** Good for knowledge, bad for working state. You don't want to "retrieve" your current task status — you want it to be immediately present.
- **Fine-tuning:** Bakes in static knowledge, doesn't help with dynamic state.
- **Vector databases:** Same as RAG — good for knowledge retrieval, not for operational state.

### The CONTEXT.md approach
None of the above solve the fundamental problem: **the agent needs to know its current obligations, not just its accumulated knowledge.**

CONTEXT.md solves this by making current obligations a first-class concept. It's not retrieval — it's a read-on-startup state file. It's not knowledge — it's operational context. It's not static — it's updated with every significant action.

The pattern is transferable to any agent system, any framework, any LLM. It requires:
1. A file the agent reads at session start
2. A discipline to update the file after significant actions
3. A format that's compact enough to read in seconds

That's it. No infrastructure. No database. No vector store. A text file.

---

## The Irony

The most sophisticated AI systems in the world — running frontier models that can reason about quantum physics and write compiler optimizations — can be derailed by forgetting a to-do item from two hours ago.

CONTEXT.md is a text file that fixes this. Under 100 lines. No dependencies. No API calls. No cost.

The hardest problems in AI infrastructure sometimes have the simplest solutions. You just have to experience the failure enough times to see the pattern.

We experienced it three times. Now we have CONTEXT.md.

> *Tweetable: "We solved AI agent amnesia with a text file. Under 100 lines. Zero cost. It's called CONTEXT.md and it survives everything the model throws away."*

---

*The agent that remembers its obligations is the agent you can trust. Memory isn't just recall — it's accountability. CONTEXT.md is how accountability survives in a system designed to forget.*
