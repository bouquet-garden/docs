# Infrastructure

*Local-first, resilient, and running 24/7 on a MacBook Pro.*

---

## Hardware

**Host:** MacBook Pro (2018), Darwin 23.6.0, x64 architecture
**Runtime:** Node.js v25.4.0 for OpenClaw Gateway
**Process Management:** macOS launchd + system crontab
**Storage:** Local SSD — SQLite databases, file-based memory, Git repositories
**Network:** Residential fiber (99.5% uptime target)
**Backup Power:** UPS for brief outages

### Why This Hardware

Dedicated hardware instead of cloud because:
- **Cost:** One-time hardware cost vs. ongoing cloud compute
- **Latency:** Local memory search and embedding generation are faster on-device
- **Privacy:** Business data on a machine Cordell physically controls
- **Simplicity:** No deployment pipelines, no cloud provider management, no infrastructure-as-code

### Known Limitations
- Power outage = full system stop (UPS provides minutes, not hours)
- Internet outage = cloud APIs fail (local fallbacks maintain memory search)
- Hardware failure = system loss (mitigated by Git-backed workspace + cloud backups)
- 2018 MacBook = adequate but not generous for heavy embedding generation

---

## The Memory Infrastructure

### Local-First Hybrid Search

```
Query
  ↓
OpenClaw Memory Search (local embeddings via node-llama-cpp)
  ↓ (fallback on failure)
Gemini API (free tier, daily quota)
  ↓
QMD Hybrid Search (BM25 text + vector via Ollama)
  ↓
Results (ranked by 70% vector + 30% text scoring)
```

### Embedding Engine

- **Provider:** Local via node-llama-cpp
- **Model:** embeddinggemma-300M (quantized Q8_0)
- **Model file:** `~/.cache/qmd/models/hf_ggml-org_embeddinggemma-300M-Q8_0.gguf`
- **Fallback:** Gemini API (auto-failover on local failure)

**Why local-first?** The February 15 Gemini outage killed cloud memory search for a day. Local inference has zero API dependency, zero rate limits, zero cost. It's slower but infinitely more reliable.

### QMD Search Engine

- **Binary:** `~/.bun/bin/qmd`
- **Index:** SQLite database at `~/.cache/qmd/index.sqlite` (224MB)
- **Hybrid scoring:** Configurable vector/text weighting (default 70/30)
- **Cache:** 50,000-entry embedding cache
- **Collections:** knowledge (1,457 files), memory (15), clawd-logs (19), clawd-knowledge (29), world-state (7)

### Ollama Service

- **Version:** 0.15.6
- **Purpose:** Local model inference for vector search and embedding generation
- **Management:** macOS launchd service (auto-start, auto-restart)

**⚠️ Operational note:** Embedding provider changes require full `launchctl stop/start` — SIGUSR1 restart does NOT re-initialize the embedding subsystem.

---

## Model Infrastructure

### Provider Configuration

| Provider | Models | Configuration |
|----------|--------|--------------|
| Anthropic | Opus, Sonnet, Haiku | Stock OpenClaw integration |
| OpenCode Zen | Kimi K2.5, GLM-4.7 | Custom provider: `https://opencode.ai/zen/v1` |
| Gemini | embedding-001 | Free tier for embeddings |
| OpenRouter | Kimi K2.5 (paid) | Fallback coding |
| Local | embeddinggemma-300M | On-device via node-llama-cpp |

### Custom Provider Example

OpenCode Zen isn't a stock OpenClaw provider. It's configured as a custom OpenAI-compatible endpoint:

```json
{
  "baseUrl": "https://opencode.ai/zen/v1",
  "models": ["kimi-k2.5-free", "glm-4.7-free", "kimi-k2.5"]
}
```

Any OpenAI-compatible API can be added this way, enabling access to new models without framework updates.

### Budget Monitoring

Frank (operations agent, Haiku) runs the budget watchdog:

| Budget % | Mode | Behavior |
|----------|------|----------|
| >50% | Normal | Full operations |
| 30-50% | Conservative | Reduce proactive checks |
| 15-30% | Austerity | Essential only, alert human |
| <15% | Emergency | Urgent alert, minimal ops |

---

## Process Management

### Gateway Service

OpenClaw Gateway runs as a launchd service:

```bash
openclaw gateway status   # Check health
openclaw gateway start    # Start
openclaw gateway stop     # Stop  
openclaw gateway restart  # Restart
```

External watchdog via launchd ensures auto-restart on crash.

### Crontab

26 active crons managed via system crontab. Each cron specifies:
- Schedule (minute, hour, day pattern)
- Model (all Haiku — hard rule)
- Agent (owner assignment)
- Timeout (default with auto-extension for slow crons)

### Health Monitoring

| Monitor | Owner | Frequency | Watches |
|---------|-------|-----------|---------|
| Budget Watchdog | Frank | 3x/day | Token spend, cascade mode |
| Cron Health | Frank | 4x/day | `consecutiveErrors >= 3` → auto-escalate model |
| Infrastructure Health | Frank | Daily | Ollama, QMD, credentials |
| Gateway Watchdog | launchd | Continuous | Process alive/dead |
| Builder Watchdog | Paired cron | Every 20 min | Progress file age |

### Auto-Recovery

- **Model escalation:** Failing crons auto-patched from Haiku → Sonnet → Opus
- **Timeout extension:** Consistently slow crons get timeout doubled (max 600s)
- **Provider fallback:** Memory search falls from local → Gemini on failures
- **Builder stall recovery:** Hung processes killed after 20 min inactivity

---

## Credential Management

### Storage

All credentials in gitignored directories:

```
~/.config/
├── notion/api_key
├── gh/hosts.yml
├── himalaya/config.toml
├── granola/mcp-tokens.json
├── bird/credentials.env
├── alice/credentials.json
├── openrouter/api_key
├── gemini/api_key
├── shipbob/pat
└── ...
```

**Documented** in gitignored `memory/credentials.md` (never committed to repo).

### Refresh Requirements

| Service | Refresh Method | Frequency |
|---------|---------------|-----------|
| Gmail (Himalaya) | OAuth auto-refresh | Occasional manual re-auth |
| Granola | `~/.config/granola/refresh-token.sh` | Periodic |
| GitHub | `gh auth` CLI | Rare |
| Others | Manual rotation | As needed |

### Security Practices

- Tool allowlists per agent (Builder can't access email credentials)
- Alice's own service accounts (separate from Cordell's)
- Audit trail via git commits
- No secrets in committed files

**Planned:** 1Password CLI for encrypted credential storage with rotation.

---

## Data Architecture

### File-Based State

```
workspace/
├── SOUL.md, AGENTS.md, TOOLS.md    # Identity and configuration
├── MEMORY.md                        # Long-term curated memory
├── CONTEXT.md                       # Current operational state
├── memory/
│   ├── YYYY-MM-DD.md               # Daily notes (ground truth)
│   ├── solutions/                   # Reusable solution files (20+)
│   └── plans/                       # Design docs
├── cache/
│   ├── world-state.md              # Business context cache
│   ├── notion-tasks.json
│   ├── email-state.json
│   └── entities.json
└── docs/                           # Documentation
```

### SQLite Databases

- **QMD Index:** `~/.cache/qmd/index.sqlite` (224MB) — search index
- **alice.db:** 8 tables for structured operational data
- **ShipBob warehouse:** Local fulfillment billing data

### Git as Backup

Workspace changes committed to GitHub regularly. Full history provides:
- Rollback to any point in time
- Audit trail of every modification
- Recovery from accidental changes
- "What did we change last Tuesday?" → `git log --since="last tuesday"`

---

## Operational Runbook

### Daily Operations (Automated)
- Morning digest at 9 AM
- Email triage 2x/day
- Fact extraction 4x/day
- Budget monitoring 3x/day
- Knowledge re-indexing nightly

### Weekly Operations (Automated)
- Sprint setup
- Weekly synthesis (daily notes → MEMORY.md)
- Cron health audit
- Obsidian vault sync

### Manual Maintenance (As Needed)
- Credential refresh (Himalaya, Granola)
- QMD index rebuild (after major content changes)
- Cron audit (disable no-ops, verify model IDs)
- AGENTS.md updates (new rules from incidents)

---

*Infrastructure cost: $0 ongoing (runs on existing hardware) + ~$3/day API usage. Total system uptime dependent on MacBook availability and internet connectivity. All critical operations have local fallbacks for cloud API failures.*

*See also: [Model Resilience](../tier2/model-resilience.md) for fallback architecture, [Infrastructure & Monitoring](../tier2/infra-monitoring.md) for monitoring deep dive.*

---

*← Back to [README](../README.md) | Next: [Deep Technical](../02-deep-technical/)*
