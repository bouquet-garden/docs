---
title: "Product Vision: Where Alice Is Going"
---

# Product Vision: Where Alice Is Going

*From custom workspace to platform. What the AI Chief of Staff becomes when it grows up.*

---

## The Current State

Alice today is a bespoke system — configuration files, cron jobs, custom scripts, and hard-won operating rules running on a MacBook Pro. It works. It works well enough that a single founder runs a creative agency with it as his most reliable team member.

But it's bespoke. Setting it up required weeks of iteration, terminal comfort, and a willingness to debug cron failures at midnight. That's fine for one operator. It doesn't scale to a hundred.

This document is about what happens when it does.

---

## The Product Layers

### Layer 1: The Companion App (In Progress)

The most immediate product need is observability. Right now, understanding what Alice is doing requires reading log files, checking cron output, and monitoring Telegram messages. The Alice Dashboard — a custom Next.js app — was the first step toward making the system legible.

**What it shows:**
- Token economy — budget tracking, usage visualization, cost attribution
- Email intelligence — triage status, thread health, ownership tracking
- Orchestration — cron status, builder progress, session management
- Knowledge infrastructure — QMD search, embedding status, memory health
- Gateway — active sessions, model routing, fallback triggers

**What it enables:**
- Builder supervision with progress bars (watching overnight builds in real-time)
- Inline task approval/rejection (Telegram buttons for one-tap decisions)
- World state viewer (what Alice knows about the business right now)
- Debugging (when something goes wrong, see the decision chain)

The dashboard isn't just monitoring. It's the beginning of a control plane — the surface through which a human conductor orchestrates AI agents.

### Layer 2: The IDE for Personal AI (Concept)

The research phase produced a concept we keep coming back to: an "IDE for personal AI agents." Not an IDE for writing code — an IDE for configuring, monitoring, and improving your AI workforce.

Think VS Code, but instead of code files, you're managing:
- **Agent definitions** — Who they are, what they do, which model they use, what tools they have access to
- **Skill library** — Browsable, installable, configurable capabilities
- **Cron scheduler** — Visual timeline of scheduled jobs with health indicators
- **Memory explorer** — Browse and search across all memory layers
- **Rule editor** — Operating standards with the incident history that spawned each one
- **Session viewer** — Watch agents work in real-time, intervene when needed

The IDE metaphor works because it captures the right relationship: you're not "using" Alice the way you use a chatbot. You're *configuring and improving* Alice the way a developer configures and improves a codebase. The workspace IS the product.

### Layer 3: The Platform (Aspirational)

If the IDE makes one person's Alice better, the platform makes everyone's Alice better.

**ClawdHub** — a marketplace for skills, agents, and configuration templates:
- Share the email triage pipeline as an installable skill
- Publish the overnight builder pattern as a reusable template
- Browse agent configurations optimized for agencies, DTC brands, consulting practices
- Community-contributed integrations (today we have 67 skills — the long tail is infinite)

**Playbook Packages** — pre-configured operational stacks:
- "AI Chief of Staff Starter" — email intelligence + sprint management + morning briefings
- "Overnight Builder Suite" — autonomous coding + stall detection + delivery guarantees
- "Knowledge Infrastructure" — 4-layer memory + vector search + fact extraction
- "Agency Operations Center" — multi-agent architecture + client management + vendor coordination

Each package represents weeks of iteration distilled into an installable configuration. The hard-won operating rules, the incident-born safety gates, the model routing optimizations — all packaged for someone who wants the outcome without the journey.

---

## The Vision Features

### Meeting Prep Briefs

**Current state:** Meeting transcripts get processed after the meeting into structured Notion tasks. That's valuable, but reactive.

**Vision:** Before every calendar event, Alice generates a prep brief:
- Who you're meeting with (relationship history, last interaction, communication preferences)
- What's active (related Notion tasks, email threads, Slack conversations)
- What changed since last meeting (new developments, completed work, blockers)
- Suggested talking points (based on current priorities and relationship context)
- Open questions from previous meetings (extracted from past transcripts)

This turns every meeting from cold start to warm start. The prep brief is generated automatically — Cordell just reads it on his way to the meeting.

### Sentiment Drift Detection

**Current state:** Alice tracks thread ownership and staleness but doesn't track emotional trajectory.

**Vision:** Natural language analysis of email and Slack threads to detect when conversations are going sideways:
- Increasing formality in previously casual threads → possible tension
- Response time increasing from a previously responsive contact → possible disengagement
- Tone shift from collaborative to transactional → possible relationship cooling
- Escalation language appearing ("going to need to," "at this point," "unfortunately") → intervention needed

The system flags these before they become problems. "Heads up: the thread with Vendor X has shifted in tone over the last three emails. You might want to get on a call."

### Predictive Escalation

**Current state:** Alice escalates things that are already problems (overdue tasks, stale threads).

**Vision:** Predict which items will become problems in 48 hours:
- Deadline approaching + no progress visible + assignee has a full calendar = likely miss
- Vendor hasn't responded + they usually respond within 24 hours + production timeline is tight = potential delay
- Client asked a question on Monday + it's Wednesday + no response drafted = relationship risk

The shift from reactive ("this is a problem") to predictive ("this will become a problem") is the difference between a competent assistant and a strategic chief of staff.

### Voice Pipeline

**Current state:** All interaction is text-based via Telegram.

**Vision:** Voice as input and output:
- **Voice notes → structured actions:** Cordell records a 2-minute voice note on his commute. Alice transcribes it, extracts action items, cross-references against current priorities, and creates tasks or drafts responses. By the time he arrives at his desk, the work is staged.
- **Audio briefings:** Morning digest as a 3-minute audio summary instead of a wall of text. Playable while making coffee.
- **Conversational interaction:** Voice-to-voice for quick decisions. "Alice, what's the status on the Portless deal?" → spoken summary with context.

The infrastructure exists (Whisper for transcription, ElevenLabs for TTS). The integration is the work.

### Self-Deployment

**Current state:** Dashboard updates require manual deployment.

**Vision:** Alice manages her own infrastructure:
- Code changes → PR created → tests pass → auto-deploy to Vercel
- Configuration changes → validated → applied to gateway → health check → rollback if broken
- New cron jobs → tested in isolation → deployed with monitoring → disabled if failure rate exceeds threshold

This is the final loop closure: the AI improves itself, deploys the improvements, validates they work, and rolls back if they don't. Human approval gates remain for anything that affects external communication or data.

### Mobile Node

**Current state:** Alice runs on a stationary MacBook Pro.

**Vision:** iPhone as a sensor and notification surface:
- Camera snap for quick document capture → OCR → action
- Location-aware reminders ("you're near the post office — pick up the package")
- Push notifications with inline action buttons
- Screen recording for debugging mobile workflows

The OpenClaw node architecture already supports paired devices. Extending to mobile makes Alice ambient — not just on the desk, but in the pocket.

---

## The Architecture That Enables This

Everything in the vision section builds on patterns that already exist in the current system:

| Vision Feature | Current Foundation |
|---|---|
| Meeting prep briefs | Granola transcript processing + Notion cross-reference |
| Sentiment detection | Thread ownership tracking + email classification |
| Predictive escalation | Staleness scoring + calendar integration |
| Voice pipeline | Whisper installed + ElevenLabs TTS available |
| Self-deployment | GitHub integration + Vercel accounts |
| Mobile node | OpenClaw node pairing architecture |

None of these require architectural changes. They're extensions of existing capabilities — new skills, new crons, new integrations layered onto the same foundation.

That's the advantage of building AI as infrastructure rather than as features: the infrastructure compounds. Every new capability benefits from the memory system, the model routing, the resilience layer, and the compound learning loop. There's no integration tax when everything already shares context.

---

## The Competitive Landscape

### What Exists Today

**Serif.ai** — Email triage and drafting. Single-surface, no cross-domain context, SaaS dependency.

**Lindy.ai** — AI assistant with workflow automation. Good UI, but workflows are pre-built templates rather than emergent from operational experience.

**AutoGPT / CrewAI / LangChain agents** — Developer tools for building agents. Powerful but require significant engineering to operationalize.

**ChatGPT / Claude** — Conversational AI. No persistent context, no autonomous operation, no multi-surface integration.

### Where Alice Differentiates

The differentiator isn't any single feature. It's three properties that are hard to replicate:

1. **Context depth** — Cross-surface intelligence that comes from integrating with everything, not just one domain.
2. **Compound learning** — Operating rules born from real incidents, accumulated over weeks of production use. This institutional knowledge can't be generated — it has to be lived.
3. **Local-first architecture** — Business data stays on your machine. No SaaS vendor has access to your emails, projects, and vendor relationships simultaneously.

The moat is time. Every week Alice runs, the institutional knowledge deepens. A new competitor starting today has to accumulate their own incidents and learnings. That takes months, and the system that's been running for months has months of additional context.

---

## The End State

> A business that runs itself, with a human conductor orchestrating AI agents.

Not fully autonomous — the human makes judgment calls, sets strategy, and maintains relationships that require human trust. But everything else — the operational machinery of running a business — handled by AI infrastructure that gets better every week.

The human focuses on the work that requires being human: creative decisions, relationship building, strategic thinking. The AI handles the work that's important but not uniquely human: triage, tracking, coordination, follow-up, reporting, scheduling, monitoring.

That's not a replacement of human work. It's an amplification of human capability. One person with the right infrastructure operates at the scale of a team, with better context, better follow-through, and perfect institutional memory.

$3/day. 24/7. Getting better every week.

That's the product.

---

*[NEEDS CORDELL INPUT]: Specific companion app timeline and Sprint 1 scope. The research documents reference a 4-phase roadmap and $39/mo pricing model — confirm if these are still current.*

---

*← Back to [README](../README.md) | Section: [The Alice Story](./)*
