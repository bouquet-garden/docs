# Unique Tools We Built

*24 custom scripts that show operational depth. Not wrappers — tools with judgment.*

---

## Why Custom Tools Matter

Every AI agent deployment eventually needs tools that don't exist. Not API wrappers — operational tools that encode business logic, handle edge cases, and integrate with the specific workflows of the business.

Alice has 24 custom scripts. Each one was built in response to a real operational need that no off-the-shelf solution could handle. Together, they represent weeks of iteration and the accumulated operational knowledge of the system.

> The scripts are the proof that this isn't a demo. Demos don't need credential health checkers or draft acceptance measurement.

---

## The Star Players

### Ralph Builder (`ralph-builder.sh`)

**What it does:** Autonomous coding loop that spawns Claude Code with enforced timeouts, progress tracking, and stall detection.

**The story:** In early February, a coding agent ran for 7 hours on a single task. The timeout variable was declared but never actually enforced — a classic "declared but not applied" bug. The agent churned through tokens on an edge case loop until morning.

**How it works:**
```bash
./ralph-builder.sh <project_dir> [task_file] [model] [max_loops] [timeout_per_loop]
```

1. Reads a `.tasks.md` file with checkboxes (`- [ ]` unchecked, `- [x]` done)
2. Spawns Claude Code to work on the next unchecked task
3. **Real timeout enforcement:** Background watchdog PID + kill signal after configurable seconds (default 300s)
4. After each loop: checks build (`npm run build`), marks task complete, moves to next
5. Progress file at `/tmp/ralph-last-progress-{hash}` for external stall detection
6. Exits when all tasks checked or max loops reached

**Key innovation:** The watchdog runs as a background process that kills the Claude Code PID if it exceeds the timeout. This is the fix for the 7-hour stall — real enforcement, not honor-system timeouts.

**Used for:** Dashboard builds, feature development, dependency updates, test coverage expansion.

---

### Commslayer CX Triage (`commslayer-triage.py`)

**What it does:** Auto-resolves CX conversations in Chatwoot by label classification. Zero AI cost — pure API calls.

**The story:** Superculture's Chatwoot inbox had accumulated 565 open conversations. Most were noise — vendor notifications, system alerts, spam. They buried genuine customer issues.

**How it works:**
1. Reverse-engineered Chatwoot's REST API by watching browser network requests
2. Fetches all open unassigned conversations page by page
3. Classifies by label: `notification`, `vendor`, `hiring` → auto-resolve
4. Keeps `customer-support` labeled conversations open
5. Uses concurrent futures for parallel resolution
6. Supports dry-run mode for safe preview

**Result:** 565 conversations resolved in 3 minutes. Cost: $0. A full-time CS rep costs $45K/year.

**Pattern worth stealing:** When a platform doesn't have a public API, open browser dev tools → Network tab → perform the action → copy the curl request → extract auth + endpoints → build automation. Works on any web app.

---

### Email Delegation Detector (`email-delegation-detector.py`)

**What it does:** Scans Cordell's inbox and sent folder for delegation patterns, enriches context, prevents duplicates, then creates Notion tasks.

**The story:** Cordell delegates to team members (Shyla, Felix) via email. Those delegations need to become tracked Notion tasks. Manually creating tasks from emails is exactly the kind of work an AI should handle.

**How it works:**
1. Scans email for action phrases: "please coordinate," "can you," "follow up on," "need you to"
2. Identifies team members by name patterns and email addresses
3. Enriches with business context (known addresses, vendor details, SOPs)
4. Checks shared task registry for duplicates (prevents re-creating existing tasks)
5. Creates Notion tasks with full context: assignee, source email, deadline inference
6. Respects business hours — skips processing outside work times unless forced

**Key innovation:** The duplicate detection uses a fingerprint system — topic keywords hashed and compared against a shared registry. This prevents the "I already have that task" problem that plagues automated task creation.

---

### Granola-Notion Bridge (`granola-notion-bridge.py`)

**What it does:** Prevents duplicate task creation when processing meeting transcripts.

**The story:** Early meeting processing created 30 tasks from a single call — half redundant with existing work. The bridge checks the shared task registry before creating anything.

**How it works:**
1. Takes an assignee, title, and topic keywords
2. Computes a fingerprint from keywords
3. Checks the shared registry for fuzzy matches
4. Returns "duplicate" or "create"
5. After creation, registers the new task for future dedup

**Used by:** The Granola meeting processor cron, which runs after every meeting to extract and track action items.

---

### Voice Note Processor (`process-voice-note.sh`)

**What it does:** Transcribes voice notes with local Whisper and extracts structured priorities.

**The story:** Inspired by Eric Siu's "voice → priority → agent action" pipeline. The idea: Cordell sends a voice note while walking or driving, and Alice extracts priorities and reprioritizes work.

**How it works:**
1. Takes an audio file path
2. Runs OpenAI Whisper locally (base model for speed on Intel hardware)
3. Outputs structured text transcription
4. Downstream: Alice parses for priorities, action items, and decisions

**Performance:** ~9 seconds per second of audio on CPU. Not fast, but free — no API call, no data leaving the machine.

---

### Credential Health Check (`credential-health-check.sh`)

**What it does:** Tests every integration and reports failures only.

**What it tests:**
1. Google Calendar OAuth
2. Gmail via gog
3. IMAP work account (himalaya)
4. IMAP alice account (himalaya)
5. GitHub SSH deploy key
6. GitHub CLI authentication
7. Notion API key
8. Obsidian vault git remote

**The story:** In W11, IMAP authentication failed silently. This cascaded: Annie couldn't escalate → Alice couldn't verify → Cordell never saw P1 payroll items aging 5+ days. The credential health check was built to catch these failures proactively.

**Design:** Reports failures only — no noise on success. Runs daily at 8 AM. If everything is healthy, you hear nothing. If something breaks, you know immediately.

---

### Draft Acceptance Measurement (`check-draft-acceptance.sh`)

**What it does:** Compares Gmail Drafts vs Sent Mail to measure how often Cordell accepts Alice's email drafts as-is.

**Edit categories:**
1. `sent_as_is` — Draft sent without changes
2. `small_factual_correction` — Numbers, dates, names fixed
3. `tone_shift` — Same info, different voice
4. `strategy_shift` — Different approach entirely
5. `full_rewrite` — Draft essentially replaced
6. `discarded` — Draft ignored, manual email sent instead
7. `pending` — Still in Drafts, not yet acted on

**Why it matters:** This is how Alice measures her own quality. A high `sent_as_is` rate means the drafts are good. A high `tone_shift` rate means Alice needs to better match Cordell's voice. A high `discarded` rate means the drafts aren't useful. This data feeds back into the compound learning loop.

---

## The Supporting Cast

### Token Tracker (`token-tracker.sh`)
Captures session stats (context percentage, queue state) to JSONL for usage analysis. Enables budget forecasting and cost attribution.

### Slack Digest (`slack-digest.sh`)
Compiles Slack activity into a digestible summary. Catches decisions, blockers, and mentions that might not surface through other channels.

### Notion Context Enrichment (`notion-context-enrichment.sh` + `notion-context-enrichment-light.py`)
Cross-references email, Slack, and calendar signals against Notion tasks. Surfaces contradictions: "Slack says done, Notion says in progress."

### Dashboard Data Collector (`collect-dashboard-data.sh`)
Aggregates data from all sources into a unified dashboard view: token economy, email intelligence, cron health, memory stats.

### Granola Auto-Cache (`granola-auto-cache.sh` + `granola-cache.py`)
Caches meeting transcripts locally for faster processing and offline access. Handles Granola MCP token refresh.

### Notion Task Registry (`notion_task_registry.py`)
Shared dedup registry that prevents any script from creating duplicate Notion tasks. Used by email delegation detector, meeting processor, and manual task creation.

### Notion Status Sync (`notion-status-sync.py`)
Keeps Notion task statuses in sync with actual progress across surfaces.

### Infrastructure Health Test (`infra-health-test.sh`)
Comprehensive system health validation — disk space, memory, process counts, network.

### Safe Search (`safe-search.sh`)
Wrapper around search tools with rate limiting and error handling.

### Markdown-Git Link Converter (`md-git-link.sh`)
Converts local file paths to GitHub URLs. Essential for Telegram delivery — local paths are useless on mobile.

---

## The Philosophy

These scripts share common principles:

1. **Dry-run by default.** Most scripts have a `--dry-run` or preview mode. Test before executing.
2. **Failures only.** Health checks report problems, not "all clear" spam.
3. **Dedup everywhere.** Before creating anything (tasks, alerts, entries), check if it already exists.
4. **Local first.** Voice processing, embedding, search — all run locally when possible.
5. **Business logic, not API wrappers.** Each script encodes judgment about when and how to use the underlying APIs.

> The scripts aren't the impressive part. The impressive part is that each one exists because a real operational need emerged that no tool on the market solved. That's what building infrastructure looks like.

---

*← Back to [README](../README.md) | Related: [Multi-Agent Orchestration](multi-agent-orchestration.md) · [Infrastructure](../01-foundations/infrastructure.md)*
