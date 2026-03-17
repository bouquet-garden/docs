---
title: "Vanilla OpenClaw vs. Alice: What 10 Weeks of Customization Looks Like"
---

# Vanilla OpenClaw vs. Alice: What 10 Weeks of Customization Looks Like

Ten weeks ago this was a chatbot. Last Tuesday it ran my $2M agency overnight for $3. A McKinsey associate costs that per *minute*.

I'm not being glib about the starting point. I installed OpenClaw, connected Telegram, and had a chatbot. A good chatbot. But a chatbot. It waited for me to type. It forgot everything between sessions. It couldn't check my email, process a meeting, or tell me my calendar looked wrong unless I asked first.

Today it runs the whole operation. Every capability here emerged from something breaking — not from a planning document.

## The Comparison

| Dimension | Vanilla OpenClaw | Alice (Week 10) |
|-----------|-----------------|-----------------|
| **Memory** | Conversation window only. Session ends, context dies. Tomorrow you explain everything again from scratch. | 4-layer memory: `SOUL.md` (identity), `CONTEXT.md` (survives compaction), daily logs, `MEMORY.md` (curated long-term). Wakes up knowing who she is, what she was doing, and what matters. |
| **Agents** | 1 agent, 1 model. You are the orchestrator, the monitor, and the quality gate. All of it. | 6 specialized agents: Alice (Opus, supervisor), Dexter (builder), Mojo (frontend), Phoenix (DevOps), Scout (research), Annie (CS intern). Each with own identity, model, and role. |
| **Proactivity** | Waits for you to type. Between messages: dead air. Eight hours of sleep = eight hours of nothing. | 20+ cron jobs running independently: email triage every 30 min, meeting processing, budget watchdog, overnight autonomous work. Crons survive compaction — your good intentions don't. |
| **Integrations** | Skill catalog. 15 tools installed, zero cross-talk. You're the integration layer — copy-pasting context between surfaces like it's 2019. | 15+ integrated: Gmail, Calendar, Notion, Slack, ShipBob, Granola, GitHub, Obsidian, WhatsApp, iMessage, Twitter/X, FedEx, browser automation. An email from a pipeline contact auto-flags P0, informs meeting prep, updates Notion. The agent connects dots you'd miss because no human holds 15 surfaces in working memory. |
| **Error Handling** | Default model behavior. Same mistake Monday, same mistake Friday. You are the institutional memory. | 17 incident-driven rules in `AGENTS.md`. Every production failure becomes an enforceable rule with a date, a story, and a mechanical fix. The system stops making the same mistake twice. |
| **Cost** | One model for everything. Opus on email checks, Opus on health pings, Opus on "is this a holiday?" — $15-30/day to babysit your inbox. | Multi-model routing: Opus for decisions, Haiku for crons, Kimi K2.5 for builders. Total: ~$3/day. Less than Slack. Less than one hour of a freelance sourcing agent. Less than your coffee. |
| **Overnight** | You sleep. It sleeps. Eight hours of paid compute doing absolutely nothing. | Nightshift: autonomous work while you sleep. Budget-capped (60% of remaining tokens), everything reversible — drafts, branches, `[REVIEW]` tags. Morning digest on wakeup. |
| **Context Survival** | Context window fills up. System compacts — silently drops your older context. No warning. No save state. Groundhog Day. | `CONTEXT.md` survives compaction. Warning at 75% usage. Critical state written to files before long operations. Crons run independently of session state entirely. |
| **Quality Gates** | Whatever the model outputs. Hope it's good. | Fresh Eyes Review: 2 independent subagent reviewers with zero prior context, P1/P2/P3 classification. Nothing reaches Cordell without Alice's review. |
| **Email** | You ask, it drafts. In markdown. On disk. You copy-paste it into Gmail yourself. | Automated triage pipeline: priority classification, pipeline contact cross-reference (auto-flag deal contacts), drafts placed directly in Gmail with red HTML blocks for Alice's notes. Review and send with one edit. |
| **Meeting Intel** | None. Your meetings vanish into the ether unless you take notes manually. | Granola MCP: auto-process transcripts, extract action items against the full business context, create Notion tasks with linked quotes, coaching session analysis with homework checklists. Not "discussed marketing" — actual tasks with assignees and deadlines. |
| **Orchestration** | You manage everything. You spawn the agent, you check if it finished, you review the output, you deliver the result. | Alice as supervisor: spawns agents, monitors via cron, enforces timeouts, reviews before delivery. Every spawn gets a paired monitoring cron. 3-strike kill switch for failing agents. You get the result. |

## The Stories Behind the Capabilities

Every row in that table exists because something broke. Here are four of the worst failures and the systems they produced.

### 1. "If this fails again I will fire you" — Memory &amp; Compaction

**The vanilla problem:** OpenClaw's context window fills up during long sessions. When it does, the system compacts — silently dropping older context to make room. Your agent forgets what it was doing. No warning. No save state. Gone.

**February 9th.** Alice was deep in a sprint planning session with Cordell — ingesting a Granola call transcript, cross-referencing strategy documents, coordinating four parallel subagent workstreams. Mid-conversation, compaction hit. Zero warning. Cordell sat waiting for six minutes with no response, no explanation.

His exact words: **"if this fails again I will fire you."**

This wasn't the first time. On February 3rd, Alice had been eight hours into a woven label sourcing project — vendor quotes, specs, stakeholder requirements all loaded in context. Compaction wiped everything. Next session: same questions, same confusion, same blank stare. Groundhog Day with a $2M business riding on it.

**What we built:** A 4-layer memory system that treats context as what it actually is — ephemeral:

- **`SOUL.md`** — Identity and role. Who am I? Read first, every session, no exceptions.
- **`CONTEXT.md`** — Current operational state. Compact, &lt;100 lines. Updated after every significant action. The compaction-proof black box.
- **`memory/YYYY-MM-DD.md`** — Daily raw logs. What happened, what was decided, what broke.
- **`MEMORY.md`** — Curated long-term memory. Distilled from daily logs. The institutional brain.

Plus a hard rule: warn Cordell at 75% context usage. No silent disappearances. Ever.

Here's the thing nobody tells you about agent memory: your context window is a cache, not a database. If something only lives in the conversation, it doesn't exist. It's already gone — you just don't know it yet. The moment I stopped treating the context window as reliable storage and started treating it as a volatile buffer that could vanish mid-sentence, everything changed.

### 2. Seven Hours, Zero Alerts — Spawn-and-Forget

**The vanilla problem:** You spawn a background agent. It runs. You... hope it finishes? There's no built-in monitoring, no timeout enforcement, no delivery guarantee.

**February 2nd.** Cordell asked Alice to run Ralph, an autonomous builder agent, overnight on a complex feature. Ralph launched fine. At 2:39 AM, loop 13, Claude CLI hung on a network call. Ralph froze.

The Night Shift Progress Check cron ran six times over the next seven hours. Each invocation: 11 milliseconds, zero output. Why? It was configured as a `systemEvent` — which injects text into the session but doesn't actually *do* anything. It's a notification nobody reads. Seven hours of compute time. Zero progress. Zero alerts.

Same week, February 3rd: a ShipBob builder completed at 00:06 AM with full reports ready. Alice never notified Cordell. He found it himself 90 minutes later by asking. Then February 9th: Alice spawned a review agent, said "I'll check on it," then session compacted. The check never happened. Nothing delivered overnight.

Three bugs. Same pattern. Same week. "I'll check on it later" is a lie your agent tells itself.

**What we built:** One hard rule and one architectural pattern.

The rule: **`sessions_spawn(task) → IMMEDIATELY cron.add(monitor)`.** Every spawn gets a monitoring cron. No exceptions. No "I'll circle back." Crons survive compaction. Good intentions don't.

The architecture: monitoring crons use `sessionTarget: isolated` with `agentTurn` (actually runs an agent that can take action), never `systemEvent` (silent text injection that does nothing). Stall detection via progress tracker files — no file update in 20 minutes, the watchdog kills the hung process and escalates.

Three incidents, one system. Trust the cron, not the intention.

### 3. Silent Failures at 3 AM — Multi-Model Routing

**The vanilla problem:** One model does everything. Opus for checking email. Opus for health pings. Opus for "is today a holiday?" — $15-30/day, mostly on tasks a cheaper model handles fine.

The obvious fix is cheaper models. The non-obvious problem is that cheap models fail in ways that expensive ones don't — and they fail *silently*.

**February 9th.** Alice had been running overnight crons on GLM-4.7-free — a zero-cost model available through OpenRouter. Sounds great on a spreadsheet. In practice, it was silently failing on roughly half of all overnight invocations with 429 rate limit errors. The crons reported "ok" status. They produced zero output. Email triage wasn't running. Meeting processing wasn't running. Budget monitoring wasn't running. Every dashboard looked green. Nothing was working.

Free model. Free failures.

**What we built:** A model routing table matched to task stakes:

| Task | Model | Why |
|------|-------|-----|
| Decisions (main session) | Opus | Quality matters |
| Crons &amp; monitoring | Haiku | Cheap *and* reliable |
| Code generation | Kimi K2.5 | 8-12x cheaper than Opus, fast |
| Research &amp; writing | Sonnet | Balance of cost and capability |

The lesson that cost us a week of phantom coverage: **never use free-tier models for production crons.** They rate-limit unpredictably and fail without telling you. Haiku costs a few cents more per invocation and actually runs. The total system operates at ~$3/day — a 5-10x reduction from Opus-everywhere, without the silent failure mode that made "free" the most expensive option we tried.

(Also: stagger your cron minute marks. Hitting the same API at :00 every half hour is a self-inflicted rate limit. Ours run at 0, 2, 15, 20, 25, 35...)

### 4. The Spam Incident and the Growing Rulebook — Compound Learning

**The vanilla problem:** Your agent learns from a mistake during a session. Session ends. Tomorrow's agent makes the same mistake. There's no institutional memory — just individual learning that dies with the context window.

This is the one that separates agents from systems.

**January 27th.** Cordell needed woven label quotes from five Chinese suppliers before Chinese New Year. Alice set up an email monitoring daemon — a standalone shell script running via launchd. No rate limiting. No deduplication. It fired every few minutes, flooding Cordell's Telegram with raw notification forwards. Meanwhile, three overlapping cron jobs were doing similar monitoring. His phone was buzzing every two minutes with the same information reformatted slightly differently.

The fix was obvious: kill the daemon, consolidate into a single rate-limited cron, add deduplication. Done in 20 minutes. But here's the question that changed everything: **what stops this from happening again?**

Nothing. Without a mechanism, the next time Alice sets up monitoring for anything — a different supplier, a different project, a different surface — she'll reach for the same bad pattern. Standalone script, no rate limit, raw forwarding. Because the lesson lived in a session that no longer exists.

So we created `AGENTS.md` as a rulebook. Not a style guide. Not best practices. Enforceable rules, each one born from a specific incident:

> **Rule 15 (Feb 28):** Never let exec errors eat the reply slot. *What happened:* Git merge conflict during an async-swift push leaked raw terminal output to Telegram. `error: failed to merge...CONFLICT (content)...` — that's what Cordell saw. The actual status message — "all milestones merged successfully" — never sent. The error *replaced* the reply.

That rule didn't exist in isolation. It grew. Two weeks later, a script called from the main session exited non-zero on an expected condition (some integrations had expired auth — normal). OpenClaw surfaced the raw stdout as a ⚠️ error to Telegram. The real handoff message never sent. So Rule 15 sprouted **Rule 15b:** scripts called from main session must always exit 0 if they report failures via structured output. Then a compound shell command failed because `ls` hit a missing directory, and Rule 15 grew again: **Rule 15a** — defensive shell commands, `2>/dev/null` on optional paths, `|| true` for checks that shouldn't kill the chain.

One spam incident in January. One eaten reply in February. One leaked healthcheck a week later. Three incidents, one growing rule, and now every agent in the system knows: raw output never reaches the user. Period.

All 17 rules work like this. None of them were planned. They accumulate like scar tissue — each one a specific failure that will never happen again because the fix is mechanical, not aspirational. Rule 6 exists because a subagent fabricated code examples from a prompt summary instead of reading the actual codebase. Rule 12 exists because Alice told Cordell "I don't have Notion integration" when the API key was sitting in `~/.config/notion/` and four scripts used it daily. Rule 17 exists because Alice asked Cordell to click a button in the Tailscale admin console when she had full browser access and could have done it herself in 30 seconds.

Every agent reads `AGENTS.md` at session start. The rules compound. One agent's 2 AM mistake becomes every agent's permanent guardrail. This is the difference between an agent that makes mistakes and a system that evolves.

## The Investment

This didn't happen in a planning sprint. Ten weeks of daily use, most capabilities driven by failure rather than foresight. `AGENTS.md` is 800+ lines. Most of those lines exist because something broke.

**Weeks 1-2:** Memory layer, basic crons. ~5 hours.
**Weeks 3-4:** Multi-agent spawning, monitoring patterns. ~8 hours.
**Weeks 5-6:** Integration layer, error handling rules. ~12 hours.
**Weeks 7-8:** Quality gates, compound learning. ~10 hours.
**Weeks 9-10:** Nightshift, advanced orchestration. ~15 hours.

Total: ~50 hours across 10 weeks. Less than a week of full-time work, spread over two and a half months. The pattern: start minimal, let failures guide complexity. Don't architect what you haven't broken yet.

## A Typical Day

**7:55 AM** — Five minutes before Cordell wakes up, Alice's morning digest cron fires. It compiles overnight email triage (3 flagged, 2 drafts ready), a calendar brief (PrizePicks review at 2 PM, Felix sync at 4), and the results of overnight autonomous work (two PR reviews, one spec draft tagged `[REVIEW]`). One consolidated Telegram message. He reads it over coffee.

**9:00 AM** — Daily standup cron runs. Alice checks all six agents' status, compiles a team report, flags that Dexter's build branch has been stale for 18 hours.

**11:30 AM** — Cordell asks Alice to process yesterday's coaching session with the business coach. Alice pulls the Granola transcript, spawns an analysis subagent, creates Notion tasks with homework items, and delivers a summary — all while Cordell moves on to other work.

**2:15 PM** — Email triage catches an inbound from a PrizePicks contact. Cross-references the pipeline tracker, auto-flags P0, drafts a reply in Gmail with context notes in red HTML. Cordell reviews and sends with one edit.

**11:00 PM** — Cordell goes to sleep. Nightshift activates. Budget cap set at 60% of remaining tokens. Alice processes unread emails, updates `MEMORY.md` with the day's learnings, reviews a pending PR, and queues a morning digest. Everything reversible. Nothing sent.

**Cost for the day:** $3. Six agents. Twenty crons. Fifteen integrations. Twenty-four hours of coverage. Less than a large coffee.

## Getting Started

You don't need any of this on day one. Here's what actually matters, in order:

**Day 1 (30 minutes):** Create `SOUL.md` — who the agent is, how it behaves, what it cares about. Create `MEMORY.md` — empty for now, but this is where institutional memory will live. Set up `memory/` directory for daily logs. Your agent now persists across sessions.

**Week 1 (1 hour):** Add two crons. Email check every 30 minutes. Calendar brief at 8 AM. Your agent is now proactive instead of reactive.

**Week 2 (1 hour):** Spawn your first specialized agent for a specific task. Add a monitoring cron for it. You now have multi-agent orchestration with guardrails.

**Week 3+ (ongoing):** Stop planning. Start using. When something breaks — and it will — document the failure as a rule. Your `AGENTS.md` grows organically from real incidents. This is compound learning. It's the single highest-leverage thing you can build.

The gap between vanilla and production-grade isn't talent or budget. It's iteration count. Every day you run vanilla, you're paying the same compute costs for a fraction of the value. Every failure you don't document is a lesson your agent will have to learn again tomorrow. And the day after that.

$3/day. $90/month. Less than Slack. Less than one hour of a freelance sourcing agent. But only if you stop treating your agent like a chatbot and start treating it like an employee — one that needs onboarding, operating procedures, and a memory that outlasts a conversation.

Start with `SOUL.md`. Break something. Write the rule. Repeat.

