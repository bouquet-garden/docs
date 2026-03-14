# Skills & Integrations

*67 skills, 10+ external services, and the integration philosophy behind them.*

---

## The Skill System

OpenClaw's skill system provides modular capabilities that agents can invoke. Think of skills as specialized tools — each handles a specific domain with its own documentation, error handling, and usage patterns.

Alice has 67 skills installed, spanning communication, development, research, infrastructure, and productivity. Most are stock OpenClaw skills from the ClawdHub ecosystem. Several are custom-built for Alice's specific workflows.

### Skill Architecture

Each skill follows the SKILL.md pattern:
```
skills/{skill-name}/
├── SKILL.md          # Documentation (purpose, examples, error handling)
├── index.js/ts       # Implementation
└── config.json       # Configuration (if needed)
```

The SKILL.md is key — it's the skill's playbook. When an agent uses a skill, SKILL.md provides context on how to use it correctly, what errors to expect, and what patterns produce the best results. This is documentation-as-prompt-engineering: better docs → better tool use → better outcomes.

---

## Core Skills by Domain

### Email Management

**himalaya** ⭐ — CLI email via IMAP/SMTP. Two accounts (work + personal). The most safety-critical skill in the system.

```bash
# Safe operations (no approval needed)
himalaya envelope list -a work -s 20          # List emails
himalaya message read -a work 12345           # Read specific email
himalaya envelope list -a work 'from kevin'   # Search by sender

# Draft (safe — saves to folder, never sends)
echo 'From: ...
To: ...
Subject: ...' | himalaya message save -a work -f "[Gmail]/Drafts"

# Send (REQUIRES EXPLICIT "send" INSTRUCTION)
echo '...' | himalaya template send -a work
```

**The Outbound Gate:** Born from the February 13 incident where Alice sent live emails when asked to "draft" responses. Hard rule: `message save` = safe, `template send` = requires explicit approval. No ambiguity in the command boundary.

**email-triage** — Automated workflow that classifies inbox into P0-P3 priorities, tracks thread ownership ("ball with us" vs "ball with them"), and prepares draft responses with contextual Alice notes.

### Calendar & Meetings

**gog** — Google Calendar access. Multi-account (work + personal). Meeting prep, scheduling, conflict detection.

**granola** — Granola MCP API for meeting transcripts. Pulls 50-70K character raw transcripts, processes into structured Notion tasks with linked quotes. Requires periodic token refresh via `~/.config/granola/refresh-token.sh`.

### Project Management

**notion** — Notion API integration. Sprint management, task creation, velocity tracking, audit. The core of Alice's Linear replacement.

**things-mac** — Apple Reminders / Things integration for quick personal task capture.

### Communication

**message** (Telegram) — Primary channel. Supports inline buttons for structured decision-making:
```
📬 New task candidate:
"Schedule call with Kevin"
[✅ Approve] [⏭ Skip] [✏️ Edit]
```

**slack** — Bot + reaction workflows. 🎫 on a Slack message → Notion task created. ✉️ → email draft prepared. ⏰ → reminder set.

**imsg** — iMessage CLI. Direct messages.

**wacli** — WhatsApp CLI. Vendor communication (especially useful for international suppliers).

### Development

**github** — `gh` CLI. PR management, repo operations, code review workflows.

**coding-agent** — Structured coding patterns for Builder and Mojo agents.

**expo-react-native** — Mobile development toolkit for the Mojo agent.

### Research & Intelligence

**oracle** — Web research with structured output.

**x-research** — Twitter/X research via Composio. Zero API cost.

**gemini** — Gemini API access for research, summarization, embeddings.

### Knowledge Management

**obsidian** — Obsidian vault integration. Git-synced, QMD-indexed, 1,457+ files searchable.

**qmd** — Hybrid search engine. Vector + BM25 text search across all indexed content.

**bear-notes** — Apple Notes integration for quick capture.

### Infrastructure

**healthcheck** — System health monitoring.

**session-logs** — Session history and analytics.

### Business Operations

**shipbob** — ShipBob fulfillment data. CSV → SQLite → SQL queries → Google Sheets pipeline for billing verification.

**commslayer** — Reverse-engineered Chatwoot REST API for CX automation. 565 conversations auto-resolved in 3 minutes.

---

## Integration Registry

Every integration with authentication details and access method:

| Service | Access | Config Path | Primary Agent |
|---------|--------|-------------|---------------|
| Notion | API key | `~/.config/notion/api_key` | Alice, Phoenix |
| GitHub | `gh` CLI | `~/.config/gh/hosts.yml` | Builder, Mojo |
| Gmail | Himalaya (OAuth) | `~/.config/himalaya/config.toml` | CS, Alice |
| Google Calendar | GOG | `~/.config/alice/credentials.json` | Alice |
| Granola | MCP API | `~/.config/granola/mcp-tokens.json` | Alice |
| Twitter/X | `bird` CLI | `~/.config/bird/credentials.env` | Alice |
| iMessage | `imsg` CLI | Local | Alice |
| WhatsApp | `wacli` CLI | Local | Alice |
| Apple Reminders | `remindctl` | Local | Alice |
| Apple Notes | `memo` CLI | Local | Alice |
| Obsidian | `obsidian-cli` | Local | Phoenix |
| ShipBob | PAT | `~/.config/shipbob/pat` | Frank |
| Slack | Bot token | Config | Alice |
| OpenRouter | API key | `~/.config/openrouter/api_key` | Fallback |
| Gemini | API key | `~/.config/gemini/api_key` | Embeddings |

**Rule 12 context:** Before claiming "I don't have access to X," check this registry AND check `~/.config/` for credentials. The rule exists because Alice once told Cordell she lacked Notion access while the API key was sitting right there.

---

## Custom Skills (Built for Alice)

### autonomous-builder
Overnight coding agent with progress tracking, stall detection, and delivery guarantees. Wraps the spawn/monitor/deliver pattern into a reusable skill.

### vendor-monitoring
Tracks supplier response times and communication patterns. Especially useful during the Accio/Alibaba sourcing workflow where vendor replies arrive during Chinese business hours.

### knowledge-graph
Entity and relationship extraction from conversations. Feeds the `entities.json` in the world state cache.

### email-triage
Classification pipeline: read inbox → categorize (P0-P3) → track thread ownership → prepare drafts with contextual notes → label in Gmail.

### morning-briefing
Consolidated daily digest: email priorities + calendar events + Notion sprint status + overnight builder results + any alerts.

---

## Integration Patterns

### The Reverse-Engineering Pattern

Some integrations don't have public APIs. The Commslayer (Chatwoot) integration was built by:

1. Opening Chatwoot in browser with dev tools → Network tab
2. Performing the desired action (resolve conversation)
3. Copying the curl request from the network panel
4. Extracting auth tokens, endpoint URLs, and request format
5. Building a Python script that replicates the requests

This pattern works for any web app: watch what the browser does → replicate it programmatically. Used for Commslayer and available as a template for future integrations.

### The Cross-Reference Pattern

Individual integrations have limited value. Cross-referencing is where the real intelligence emerges:

- **Email + Notion:** "This email from Kevin relates to Notion task #4521"
- **Slack + Notion:** "This Slack thread mentioned a task that isn't captured in the sprint"
- **Granola + Notion:** "The coaching call mentioned 3 action items — creating tasks with linked quotes"
- **Email + Calendar:** "You have a meeting with this person tomorrow — here's the latest email thread"

The world state cache enables this by materializing data from all surfaces into a single queryable view.

### The Safety Pattern

All external integrations follow the same safety philosophy:

- **Reading is always safe.** Check email, list tasks, view calendar — no approval needed.
- **Internal writes are safe.** Save to drafts, create draft tasks, update internal files.
- **External sends are gated.** Send email, post to Slack, push to GitHub — requires explicit approval.

This hierarchy was formalized after the February 13 email send incident and applies to every integration.

---

## Skill Creation (Custom)

When a workflow is repeated frequently enough, it becomes a skill:

1. **Identify the pattern** — What steps get repeated? What errors occur?
2. **Create skill directory** — `skills/{name}/`
3. **Write SKILL.md** — Complete docs with examples and error handling
4. **Implement** — Script or structured prompt
5. **Test with target agent** — Realistic scenarios
6. **Add to agent config** — Make discoverable

**When NOT to create a skill:**
- One-off operations → handle inline
- Simple one-liner commands → use exec directly
- Agent-specific logic → put in agent's SOUL.md

---

*67 skills installed. 10+ external integrations. The skill count grows as new workflows stabilize into repeatable patterns. Each skill represents a capability that any agent can invoke — the shared vocabulary of what Alice can do.*

*See also: [Architecture Overview](architecture.md) for system context, [Workflows & Crons](workflows.md) for how skills combine into automated workflows, [Skill Creation Playbook](../tier4/skill-creation.md) for the step-by-step guide.*

---

*← Back to [README](../README.md) | Next: [Deep Technical](../02-deep-technical/)*
