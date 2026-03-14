---
title: "Architecture Overview"
---

# Architecture Overview

*How Alice is built — and why every architectural choice traces back to a real problem.*

---

## System Topology

Alice runs on a single MacBook Pro (2018, Darwin 23.6.0, x64). This isn't a cloud deployment with managed services — it's a local-first system where the physical machine IS the infrastructure.

```
MacBook Pro (24/7 server)
    ├── OpenClaw Gateway (Node.js v25.4.0)
    ├── Alice Main Agent (Opus)
    ├── 8 Specialized Agents (distributed across models)
    ├── Local Memory Search (node-llama-cpp, embeddinggemma-300M)
    ├── QMD Index Engine (BM25 + Vector hybrid, 224MB SQLite)
    ├── Ollama Service (v0.15.6, local embedding generation)
    └── 26 Scheduled Cron Jobs (crontab)
```

### Why Dedicated Hardware (Not Cloud)

Cloud hosting was evaluated and rejected for three reasons:

1. **Cost predictability.** A Mac Mini or MacBook Pro is a one-time cost. Cloud compute is variable — and with 26 crons running throughout the day, the hourly charges add up.
2. **Local-first architecture.** Memory search, embedding generation, and the knowledge base all run locally. Pushing these to the cloud adds latency, network dependency, and cost.
3. **Data sovereignty.** Business emails, vendor relationships, client details, and operational context stay on a machine Cordell physically controls. No SaaS vendor has access to the full business context.

### Key Directories

```
/Users/admin/.openclaw/workspace/          # Alice main workspace (SOUL.md, AGENTS.md, memory/)
/Users/admin/.openclaw/                    # System config (openclaw.json, gateway settings)
/Users/admin/clawdbot/skills/              # Skill implementations (67 skills)
/Users/admin/.config/                      # Credential storage (Notion, GitHub, Gmail, etc.)
/Users/admin/.cache/qmd/                   # Search index (224MB) + local embedding model
```

Each agent has an isolated workspace (`~/.openclaw/workspace-{agent}/`) to prevent context pollution between roles.

---

## The Three Core Subsystems

### 1. Agent Orchestration

9 agents with specialized roles, models, and tool access:

| Agent | Model | Memory Search | Role |
|-------|-------|---------------|------|
| Alice | Opus | ✅ | Supervisor, user-facing, strategic |
| Builder | Kimi K2.5 → Sonnet → Opus | ❌ | Web development |
| Mojo | Kimi K2.5 → Sonnet → Opus | ❌ | Mobile development |
| Phoenix | Sonnet | ✅ | Infrastructure, documentation |
| Scout | Sonnet | ✅ | Research, intelligence |
| Frank | Haiku | ❌ | Operations, budget watchdog |
| CS | Kimi K2.5 → Sonnet | ❌ | Customer support |
| Comms | Sonnet | ❌ | Email, Slack handling |
| Cortex | Sonnet | ✅ | Strategic analysis |

**Why this structure?** Cost optimization and context isolation. Running everything on Opus costs $8-12/day. Routing email classification to Haiku ($0.001/run) and coding to Kimi K2.5 (~$0.50/hour) drops daily cost to ~$3. Memory search is disabled for agents that work from explicit specifications (Builder, Mojo) because it adds cost without adding value for task-focused work.

**Model fallback chains** ensure continuity: if Kimi K2.5 hits rate limits, the gateway routes to Sonnet automatically. If Sonnet is down, Opus takes over. This was our own innovation — born from Kimi free tier rate limits during overnight builder sessions.

### 2. Memory Architecture

Four-layer system for persistent context:

```
Layer 1: Daily Notes (memory/YYYY-MM-DD.md)
         Ground truth. Events, decisions, lessons. Never rewritten.

Layer 2: Long-Term Memory (MEMORY.md)
         Curated knowledge. Updated weekly via synthesis cron.

Layer 3: QMD Vector Database (1,457+ files indexed)
         Semantic search. 70% vector + 30% BM25 text hybrid.

Layer 4: Local Embedding Fallback (embeddinggemma-300M)
         On-device inference. Zero API cost. Zero rate limits.
```

**Why four layers?** Each was added after a specific failure:
- Daily notes existed from day one (basic logging)
- MEMORY.md was added when daily notes became too verbose to read every session
- QMD was added to enable semantic search across the Obsidian vault
- Local embedding fallback was built on February 15 after Gemini free tier rate limits killed memory search for an entire day (the API returned empty arrays with 200 status codes — silent failure)

**Search path (resilience):**
1. OpenClaw memory_search (local embeddings) → 2. QMD BM25 text → 3. QMD vector (Ollama) → 4. Direct file read

### 3. Cron System

26 automated crons organized by domain:

| Domain | Count | Examples |
|--------|-------|---------|
| Email | 4 | Triage, fast poll, follow-up tracking |
| Notion | 3 | Sprint setup, audit, task digest |
| Slack | 2 | Reactions, activity digest |
| Knowledge | 3 | Embedding index, fact extraction, synthesis |
| Monitoring | 3 | Budget watchdog, vendor replies, CX |
| Hygiene | 2 | Vault sync, overnight check |

**Scheduling:** Staggered across minute marks (0, 2, 15, 20, 25, 35, 40, 45, 50) to avoid API rate limit conflicts. Human-aware timing: morning digest at 9 AM, EOD review at 6 PM, overnight checks at 4 AM.

**All crons run on Haiku.** Hard rule: never use free-tier models for production crons. GLM-4.7-free silently failed ~50% of overnight crons before this was established. The cheapest model that actually works is infinitely cheaper than one that silently fails.

---

## Model Resilience

The 4-tier model structure:

| Tier | Model | Use |
|------|-------|-----|
| Premium | Opus | Strategic decisions, user conversations |
| Standard | Sonnet | Analysis, documentation, research |
| Economy | Haiku | Crons, simple operations, monitoring |
| Free/Cheap | Kimi K2.5 | Code development, experimental work |

**Budget cascade** protects against cost overruns:
- Normal (>50%) → full operations
- Conservative (30-50%) → reduce proactive checks
- Austerity (15-30%) → essential only, alert human
- Emergency (<15%) → urgent alert, minimal operations

**Provider diversity** prevents single-provider dependency:
- Anthropic (Opus, Sonnet, Haiku) — primary
- OpenCode Zen (Kimi K2.5, GLM-4.7) — coding, custom provider
- Gemini (embedding-001) — vector embeddings, free tier
- OpenRouter (Kimi K2.5 paid) — fallback
- Local (embeddinggemma-300M) — on-device fallback

---

## Communication Architecture

**Primary channel:** Telegram — real-time conversation with Cordell.

**Email:** Himalaya CLI with two accounts (work + personal). Safety-first: `message save` to drafts is always safe; `template send` requires explicit "send" instruction. Born from the February 13 incident where "draft" was misinterpreted as "send."

**Integrations:** 10+ external services (Notion, Gmail, Slack, Granola, GitHub, ShipBob, Obsidian, Google Calendar, Twitter/X, iMessage, WhatsApp, Apple Reminders).

**World state cache:** Unified business context refreshed periodically:
```
cache/
├── world-state.md       # Readable summary
├── notion-tasks.json    # Task data
├── email-state.json     # Thread ownership
├── slack-messages.json  # Channel activity
└── entities.json        # People, projects, relationships
```

All agents read from cache (instant, free, consistent) rather than querying live APIs (200-800ms per Notion call, token-expensive JSON parsing, inconsistent across agents).

---

## Security Model

**Credential isolation:** All secrets in gitignored `~/.config/` directories. Documented in gitignored `memory/credentials.md`.

**Tool allowlists per agent:** Builder can't send messages. Frank can't write code. CS can't modify the workspace. Each agent has only the tools its role requires.

**Alice's own identity:** Separate email, phone, GitHub, and API accounts. Can provision her own tools without Cordell's credentials.

**Outbound gate:** All external communication (email, messages) requires explicit approval. Drafts are safe; sends are gated.

**Git audit trail:** Workspace changes committed to GitHub. Full history of every modification, recoverable to any point.

---

## What's Stock OpenClaw vs. Custom

| Component | Stock OpenClaw | Custom for Alice |
|-----------|---------------|------------------|
| Agent definitions | ✅ | Enhanced with fallback chains, memory per-agent |
| Skill system | ✅ | 67 skills (mix of stock and custom) |
| Cron scheduling | ✅ | 26 crons with staggered scheduling, model optimization |
| Telegram integration | ✅ | Inline buttons, structured feedback |
| Session management | ✅ | Spawn/monitor pattern, structured handoffs |
| Memory search | ✅ | 4-layer system with local fallback |
| World state cache | ❌ | Fully custom |
| Email intelligence | ❌ | Fully custom (thread ownership, Outbound Gate) |
| Overnight builders | ❌ | Fully custom (stall detection, watchdog, delivery guarantee) |
| Compound learning | ❌ | Fully custom (incident → rule → enforcement loop) |
| Budget cascade | ❌ | Fully custom |
| CX automation | ❌ | Fully custom (reverse-engineered Chatwoot API) |

---

*This architecture evolved over 7 weeks from a single agent connected to Telegram to a 9-agent team with 67 skills, 26 crons, and a 4-layer memory system. Every component traces back to a specific operational need or failure.*

*See also: [Model Resilience](../tier2/model-resilience.md), [Memory System](../tier2/memory-system.md), [Multi-Agent Orchestration](../tier2/multi-agent.md), [Infrastructure & Monitoring](../tier2/infra-monitoring.md)*

---

*← Back to [README](../README.md) | Next: [Deep Technical](../02-deep-technical/)*
