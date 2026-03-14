---
title: "Agent System"
---

# Agent System

*9 agents, 4 model tiers, and the delegation economics that make it work.*

---

## The Agent Roster

Alice's agent team evolved from a single agent (January 2026) to a 9-agent roster through iterative expansion and a deliberate consolidation in February's v2.0 "Mission Control" release.

### Why Multiple Agents?

Three forces drive multi-agent architecture:

1. **Cost.** Email classification on Opus ($0.05/run) produces identical results to Haiku ($0.001/run). Running everything on one premium model is a 50x overspend on commodity tasks.

2. **Context isolation.** A builder generating 500 lines of code pollutes the main session context. Every subsequent interaction pays for those 500 lines in context tokens. Separate workspaces prevent this.

3. **Parallelism.** Alice can supervise while Builder codes, while Frank monitors budget, while CS handles support tickets. Sequential processing creates bottlenecks.

### The Roster

| Agent | ID | Model | Memory | Role |
|-------|-----|-------|--------|------|
| **Alice** | main | Opus | ✅ Full | Supervisor, user-facing, strategic orchestration |
| **Builder** | builder | Kimi K2.5 → Sonnet → Opus | ❌ | Web development, API design |
| **Mojo** | mojo | Kimi K2.5 → Sonnet → Opus | ❌ | Mobile/React Native development |
| **Phoenix** | phoenix | Sonnet | ✅ | Infrastructure, documentation, analysis |
| **Scout** | scout | Sonnet | ✅ | Research, intelligence gathering |
| **Frank** | frank | Haiku | ❌ | Operations monitoring, budget watchdog |
| **CS** | cs | Kimi K2.5 → Sonnet | ❌ | Customer support (Commslayer) |
| **Comms** | comms | Sonnet | ❌ | Email and Slack handling |
| **Cortex** | cortex | Sonnet | ✅ | Strategic analysis |

### Workspace Isolation

Each agent has its own workspace directory:
```
~/.openclaw/workspace/           # Alice
~/.openclaw/workspace-builder/   # Builder
~/.openclaw/workspace-mojo/      # Mojo
~/.openclaw/workspace-phoenix/   # Phoenix
...
```

This prevents context pollution: Builder's experimental code doesn't appear in Alice's workspace. Memory search (when enabled) is scoped to the agent's workspace. Tool access is configured per-agent.

---

## Model Routing

### The Tier System

| Tier | Model | Cost | Use Case |
|------|-------|------|----------|
| Premium | Opus | $$$ | Judgment, strategy, user conversations |
| Standard | Sonnet | $$ | Analysis, documentation, research |
| Economy | Haiku | $ | Crons, monitoring, simple operations |
| Free/Cheap | Kimi K2.5 | ¢ | Code generation, experimental work |

### Fallback Chains

Agents don't have a single model — they have a chain. When the primary model fails (429 rate limit, 5xx error, timeout), the gateway automatically routes to the next model:

```
Builder:  Kimi K2.5 (free) → Sonnet → Opus
Mojo:     Kimi K2.5 (free) → Sonnet → Opus
CS:       Kimi K2.5 (free) → Sonnet
Alice:    Opus (no fallback — quality non-negotiable)
Frank:    Haiku (no fallback — tasks are simple enough for any model)
```

**Why this is our innovation:** Neither Bhanu Teja's Mission Control nor Eric Siu's 14-agent system mention model fallback chains. This pattern was born from Kimi K2.5 free tier rate limits during overnight builder sessions — builders would stall completely with no recovery path.

### Memory Search Economics

Memory search is expensive: embedding generation + vector database queries. Not every agent needs it:

- **Enabled** for Alice, Phoenix, Scout, Cortex — strategic and research roles that benefit from historical context
- **Disabled** for Builder, Mojo, Frank, CS, Comms — task-focused roles that work from explicit specifications

---

## Delegation Patterns

### The 2-Call Rule

Before doing non-strategic work, Alice asks: "Is this more than 2 tool calls?"

- **>2 calls:** Spawn a specialist (cheaper model handles the work, context stays isolated)
- **≤2 calls:** Do it yourself (spawning overhead costs more than the task)

### The Spawn Protocol

```
1. Define task clearly (specify what, not how)
2. Select agent based on task type (code → Builder, research → Scout, etc.)
3. Spawn session with task specification
4. IMMEDIATELY create monitoring cron (Rule 5 — hard rule, no exceptions)
5. Monitor via progress files
6. Receive structured handoff on completion
```

**Rule 5** is the most consequential rule in the system. Three identical spawn-and-forget failures (Feb 2, 3, 9) proved that Alice's promise to "check on it" doesn't survive session compaction. The monitoring cron does — it's independent of Alice's context.

### Structured Handoffs

Every subagent completion must end with:

```markdown
## HANDOFF
### Done: [specific deliverables with file paths]
### Remaining: [next steps with enough context for a fresh agent]
### Decisions Made: [what was decided and WHY]
### Uncertain: [risks, unknowns to investigate]
```

This template exists because an overnight builder once delivered a celebratory summary ("I completed everything!") with no actionable details. The next session had to re-read all the code to determine actual state.

---

## Decision Autonomy Framework

### The Problem

Without clear boundaries, two failure modes emerge:
1. **Paralysis:** Agent waits for permission on trivial decisions
2. **Overreach:** Agent makes consequential decisions without approval (e.g., declining all backup vendors)

### The Three Tiers

**Alice (Supervisor):** Full operational autonomy. Escalates to Cordell on external communication, financial decisions, and strategy changes.

**Builder/Mojo (Leads):** Broad autonomy within known patterns. Escalate to Alice for cross-domain decisions. Escalate to Cordell for product direction changes.

**Frank/CS/Comms (Specialists):** Narrow domain autonomy. Escalate to Alice for anything outside routine. Never make decisions affecting other agents.

Inspired by Netflix's "Highly Aligned, Loosely Coupled" model, calibrated through operational experience.

---

## Agent Personality Architecture

Each agent has a personality specification:

```
SOUL.md       → Who the agent is (voice, expertise, boundaries)
BOOTSTRAP.md  → Initial setup (run once, then deleted)
AGENTS.md     → Operating rules (safety, coordination, lessons learned)
```

**Why personality matters:** Agents with clear role definitions produce more focused output. Builder writes different code when it knows it's a "lead web developer" vs. a generic agent told to "write code." The personality constrains the approach to match the specialization.

The framework critique asked "What if personality doesn't matter?" We tested it. It does — role clarity measurably improves output quality for specialized tasks.

---

## The Evolution Story

**January 2026:** 1 agent (Alice), 1 model (Opus), 1 channel (Telegram). Simple chatbot.

**Early February:** Expanded to 8 agents. Too many — n² coordination complexity (28 possible pairs). Agents stepping on each other.

**February 9 (v2.0):** Consolidated to 4 core + 3 on-demand. Merged redundant agents. Added fallback chains, budget monitoring, structured handoffs.

**March 2026:** 9 agents, stable roster. Each agent justified by clear specialization and cost optimization. Total coordination overhead: ~5-10% of budget, paid for many times over by model routing savings.

### Key Decisions Made

| Decision | Options | Chosen | Why |
|----------|---------|--------|-----|
| Alice model | Opus vs Sonnet | Opus | Coordination quality matters — Alice touches every decision |
| Team size | 3 vs 5 vs 8 | 4 core + 3 on-demand | Balance of coverage and coordination overhead |
| Budget monitor | Alice owns vs dedicated agent | Frank (Haiku) | Independent auditor, avoids Alice bottleneck |
| Agent personality | With vs without | With | Role clarity improves output quality |
| Memory per agent | All vs selective | Selective (4/9) | Cost optimization — 5 agents don't need semantic search |

---

## The 3-Strike Rule

When an agent fails the same task repeatedly:

1. **Strike 1:** Retry with adjusted parameters
2. **Strike 2:** Retry with escalated model
3. **Strike 3:** STOP. Write up what's failing. Escalate to Cordell.

Don't burn tokens hoping attempt #4 works. The write-up forces articulation of the real issue, which usually reveals a fundamental problem (wrong model for the task, missing context, bad prompt design).

---

*See also: [Multi-Agent Orchestration](../tier2/multi-agent) for deep technical details, [Architecture Overview](architecture) for system context, [Compound Learning](../tier2/compound-learning) for how agent failures become permanent improvements.*

---

*← Back to [README](../README) | Next: [Deep Technical](../02-deep-technical/)*
