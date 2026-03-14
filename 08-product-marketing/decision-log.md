---
title: "Decision Log"
---

# Decision Log

**Architectural Decision Records for the Alice System.**

Every system is the sum of its decisions. Most teams make decisions and forget why. We write them down. When future-us questions a choice, this document answers before anyone has to re-litigate.

These aren't theoretical trade-offs. Each decision was made under real constraints — budget, time, hardware, one operator with no engineering team — and validated against production results.

---

## ADR-001: OpenClaw Over Building From Scratch

**Date:** January 2026
**Status:** Active
**Context:** Needed an agent framework to run Alice. Options: build custom framework, use LangChain/CrewAI/AutoGen, or use OpenClaw.

### Options Considered
| Option | Pros | Cons |
|--------|------|------|
| **Custom framework** | Total control, no dependencies | 3-6 months to reach MVP, requires engineering team |
| **LangChain/CrewAI** | Large community, many examples | Cloud-first, opinionated architecture, abstraction overhead |
| **OpenClaw** | Local-first, cron-native, multi-agent out of the box | Smaller community, newer project |

### Decision
**OpenClaw.** Not close.

### Rationale
The operator building Alice is a creative agency founder, not a framework engineer. Building from scratch would have consumed all available time on plumbing instead of workflows. LangChain and similar frameworks optimize for cloud deployment and developer experience — not for a solo operator running agents on a laptop.

OpenClaw's killer features for our use case:
- **Local-first:** Everything runs on one MacBook Pro. No cloud bills, no latency, no data leaving the machine.
- **Cron-native:** 47+ scheduled jobs are first-class citizens, not bolted on.
- **Multi-agent orchestration:** Spawn, monitor, and coordinate agents without writing orchestration code.
- **Telegram integration:** Operator interface out of the box.

### Consequences
- Dependent on OpenClaw's development roadmap
- Smaller community means fewer Stack Overflow answers
- Local-first means hardware is a constraint (Intel MacBook Pro, not a cloud GPU cluster)

### Validation
10 weeks of production use. 47+ crons. 4 active agents. $3/day. The framework hasn't been the bottleneck — our own decisions have.

> *Tweetable: "I didn't build my AI agent framework. I built my AI agent's workflows. That's the difference between an engineer and an operator."*

---

## ADR-002: Local-First Over Cloud

**Date:** January 2026
**Status:** Active
**Context:** Where does the agent system run? Cloud VM, managed service, or local hardware?

### Options Considered
| Option | Monthly Cost | Latency | Privacy | Reliability |
|--------|-------------|---------|---------|-------------|
| **AWS/GCP VM** | $50-200 | Low (cloud-to-API) | Data in cloud | High (managed) |
| **Managed agent service** | $100-500 | Variable | Data with provider | Provider-dependent |
| **Local MacBook Pro** | $0 (owned hardware) | Low (local-to-API) | Data never leaves machine | Hardware-dependent |

### Decision
**Local.** Run everything on a dedicated MacBook Pro (Intel 2018, repurposed).

### Rationale
1. **Cost:** Zero marginal infrastructure cost. The laptop was already owned and sitting unused.
2. **Privacy:** Client data, vendor communications, financial information — none of it leaves the machine. No cloud provider's terms of service to worry about.
3. **Simplicity:** No Docker, no Kubernetes, no deployment pipelines. `git pull && restart`. The entire system is files on a filesystem.
4. **Latency:** Local file reads are &lt;1ms. Local SQLite queries are &lt;1ms. The only network calls are to model APIs, which are unavoidable regardless of deployment.

### Trade-offs Accepted
- **No redundancy:** If the laptop dies, the system is down until we set up another one. Acceptable risk for a solo operation.
- **CPU constraints:** Intel 2018 MacBook Pro isn't fast. Whisper transcription takes ~9 seconds per second of audio. Acceptable — we're not running models locally.
- **No auto-scaling:** When token usage spikes, we can't spin up more capacity. We budget-cap instead.

### Validation
Zero downtime incidents from hardware in 10+ weeks. The system runs 24/7 with negligible resource usage (agents are mostly waiting between cron triggers). Local-first turned out to be more reliable than cloud would have been — no region outages, no billing surprises, no cold starts.

> *Tweetable: "Our AI agent infrastructure runs on a 2018 MacBook Pro. Monthly server cost: $0. Sometimes the boring architecture is the right one."*

---

## ADR-003: Telegram as Primary Surface

**Date:** January 2026
**Status:** Active (until Companion App)
**Context:** The operator needs an interface to interact with Alice. Where?

### Options Considered
| Option | Availability | Rich UI | Mobile | Setup Complexity |
|--------|-------------|---------|--------|-----------------|
| **Telegram** | Always on, any device | Basic (text, buttons, images) | Native | Zero (OpenClaw built-in) |
| **Custom web app** | Requires hosting | Full control | Responsive | High |
| **Slack** | Always on | Richer than Telegram | Native | Moderate |
| **CLI only** | Terminal required | None | SSH | Zero |

### Decision
**Telegram.** It's where the operator already lives.

### Rationale
The best interface is the one you already have open. Cordell uses Telegram daily. OpenClaw has native Telegram integration. The setup was: create a bot, add a token, done.

Telegram gives us:
- **Push notifications:** Morning briefings arrive like messages from a colleague.
- **Inline buttons:** Feedback loops (✅ Looks Good / 📝 Reprioritize / 🚨 Urgent) with one tap.
- **Voice notes:** Send audio, get structured tasks back.
- **Group topics:** Organize different agent outputs into topic threads.
- **Universal access:** Phone, tablet, desktop — same interface everywhere.

### Limitations Accepted
- No rich dashboards or data visualization
- Message formatting limited to Telegram's markdown subset
- No custom UI components (drag-and-drop, kanban, etc.)
- Rate limits on bot API (not yet a problem)

### Future
The OpenClaw Companion App will eventually replace Telegram as the primary surface. But Telegram will remain as a fallback and notification channel. The lesson: start with what works, upgrade when you've earned the complexity.

> *Tweetable: "Our AI agent's interface is Telegram. Not a custom app. Not a dashboard. A chat app we already had open. The best tool is the one you'll actually use."*

---

## ADR-004: Opus for Main / Haiku for Crons

**Date:** February 2026
**Status:** Active
**Context:** Multiple AI models available at different price/quality tiers. How to allocate?

### The Model Routing Matrix
| Task Type | Model | Cost/1K tokens | Rationale |
|-----------|-------|----------------|-----------|
| **Alice (coordination)** | Claude Opus | ~$0.075 | Every coordination decision matters. Bad judgment compounds. |
| **Builders (code/design)** | Sonnet / Kimi K2.5 | ~$0.015 | Good enough for implementation, fallback chain for rate limits. |
| **Crons (monitoring)** | Claude Haiku | ~$0.001 | Simple tasks: check email, scan Slack, update state. Cheap and reliable. |
| **Embeddings/search** | Gemini free tier | $0 | Semantic search doesn't need frontier models. |

### Decision
**Tiered model routing.** Opus where judgment matters. Haiku where execution is simple. Free tier where quality is sufficient.

### Rationale
Not all tokens are created equal. A cron that checks "any new emails?" doesn't need Opus-level reasoning. But Alice deciding whether to escalate a vendor issue to Cordell or handle it autonomously — that's a judgment call worth paying for.

The math: 47 crons running on Opus would cost ~$15/day. On Haiku, they cost ~$0.50/day. Alice on Opus adds ~$2/day. Total: ~$3/day. The 30x cost difference between Opus-everywhere and tiered routing is the difference between "interesting experiment" and "production-viable business tool."

### The GLM-Free Lesson
We tried running crons on GLM-4.7-free (zero cost). ~50% failed silently with rate limit errors. Free models are for experiments, not production. Haiku at $0.001/run is infinitely cheaper than a free model that silently fails.

### Validation
$3/day average over 10+ weeks. Zero quality incidents from Haiku crons — they do simple work and do it reliably. Opus decisions for Alice have prevented multiple escalation failures that cheaper models would have missed.

> *Tweetable: "Opus for thinking. Haiku for checking. Free tier for searching. $3/day. Model routing is the unsexy superpower of production AI."*

---

## ADR-005: 4 Core Agents, Not 8

**Date:** February 9, 2026
**Status:** Active
**Context:** Original plan specified 8 full-time agents. Framework critique questioned whether this was necessary.

### The Math That Changed Our Mind
- 8 agents = 28 coordination pairs (n × (n-1) / 2)
- 4 agents = 6 coordination pairs
- Coordination overhead scales quadratically; capability scales linearly

We were spending more tokens on agents talking to each other than on agents doing work.

### Decision
**4 core agents (Alice, Dexter, Mojo, Phoenix) + 3 on-demand (Scout, Frank, Annie).**

### Merge Decisions
| Original | Merged Into | Rationale |
|----------|-------------|-----------|
| Cortex (analytics) | Phoenix (infra) | Analytics and infrastructure share tooling; separate agent was unjustified overhead |
| Comms (communications) | Alice (coordination) | Communication decisions are coordination decisions; separating them created handoff friction |

### The On-Demand Pattern
Scout, Frank, and Annie don't run continuously. They wake up for scheduled crons, do their specific task, and go back to sleep. This gives us their specialized capabilities without the coordination overhead of full-time agents.

Frank (budget watchdog) is particularly well-suited to this pattern — an auditor should be independent, not embedded in the team they're auditing.

### Validation
Sprint velocity improved after the consolidation. Fewer "who handles this?" delays. Fewer context-sharing failures between agents. The 4-agent core can handle everything we've thrown at it.

> *Tweetable: "We cut our AI team from 8 agents to 4. Productivity went up. Turns out AI teams have the same coordination overhead as human teams."*

---

## ADR-006: Notion Over Linear for Sprint Management

**Date:** February 2026
**Status:** Active
**Context:** Need a sprint management tool that both humans and agents can read/write.

### Options Considered
| Tool | API Quality | Human UX | Agent UX | Cost |
|------|-----------|----------|----------|------|
| **Notion** | Good (REST API, databases) | Excellent (flexible views) | Good (structured queries) | Free tier sufficient |
| **Linear** | Excellent (GraphQL) | Good (opinionated) | Excellent (structured) | $8/user/month |
| **GitHub Issues** | Good (REST + GraphQL) | Basic | Good | Free |
| **Plain files** | N/A | Poor (no dashboards) | Excellent (direct read/write) | Free |

### Decision
**Notion.** Already in use for business operations. API access configured. Agent can read and write tasks, update statuses, create structured databases.

### Rationale
1. **Already there:** Cordell already manages business operations in Notion. Adding sprint management to the existing workspace is zero friction.
2. **Human-friendly:** Notion's views (kanban, calendar, table) make sprint status visible without asking Alice.
3. **Agent-friendly:** The API is straightforward for CRUD operations on database entries. Agent creates tasks with properties (status, assignee, priority, deadline) that map cleanly to sprint management concepts.
4. **Single source of truth:** Agent syncs Notion state with local files via cron. Notion is the human dashboard; local files are the agent's working copy.

### Why Not Linear?
Linear is objectively better for engineering sprint management. But we're not an engineering team. We're an operator running a creative agency with AI assistance. Notion's flexibility (mixing sprint tasks with vendor databases, meeting notes, and product specs) matches our workflow better than Linear's opinionated structure.

### Validation
Bi-directional sync works. Cordell sees sprint status in Notion without opening a terminal. Alice reads sprint context before making prioritization decisions. The tool serves both humans and agents without either compromising.

> *Tweetable: "We use Notion for AI sprint management. Not because it's the best sprint tool — because it's where the human already works. Don't make the human adapt to the AI's preferred tools."*

---

## ADR-007: Why We Document Decisions

**Date:** February 9, 2026
**Status:** Permanent
**Context:** This document exists because of a specific failure pattern.

### The Pattern
1. Make a decision under pressure (migration, consolidation, architecture change)
2. Two weeks later, question why we made that decision
3. Re-research the same options
4. Either re-make the same decision (wasting time) or make a different one (creating inconsistency)
5. Repeat

### The Fix
Write it down when you decide. Include:
- What options you considered
- Why you chose what you chose
- What you're explicitly accepting as a trade-off
- How you'll know if the decision was wrong

### Why This Matters for AI Systems
AI agents get compacted. Their context windows fill up and old decisions get summarized away. An ADR document survives compaction. When future-Alice questions why we use Notion instead of Linear, she reads ADR-006 instead of re-researching it.

This is the meta-decision: **document your decisions so your AI doesn't re-litigate them.** It's the same principle as CONTEXT.md (see `compaction-survival.md`) applied to architectural choices.

> *Tweetable: "The most expensive thing in AI operations isn't compute. It's re-making decisions you already made because nobody wrote down why."*

---

## Decision Summary

| # | Decision | Date | Key Rationale |
|---|----------|------|---------------|
| ADR-001 | OpenClaw over custom/LangChain | Jan 2026 | Operator, not engineer. Local-first. Cron-native. |
| ADR-002 | Local-first over cloud | Jan 2026 | Zero cost. Total privacy. Radical simplicity. |
| ADR-003 | Telegram as primary surface | Jan 2026 | Already open. Push notifications. Good enough until Companion. |
| ADR-004 | Opus/Haiku/free tier routing | Feb 2026 | $3/day vs $15/day. Quality where it matters, cheap where it doesn't. |
| ADR-005 | 4 core agents, not 8 | Feb 2026 | Quadratic coordination overhead. Fewer agents, better results. |
| ADR-006 | Notion over Linear | Feb 2026 | Human already there. Flexibility over structure. |
| ADR-007 | Document all decisions | Feb 2026 | Prevent re-litigation across compacted sessions. |

---

*These decisions aren't permanent. They're permanent until the evidence says otherwise. The ADR format means we'll know exactly what evidence would change our minds.*
