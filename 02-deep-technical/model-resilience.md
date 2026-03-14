---
title: "Model Resilience Architecture"
---

# Model Resilience Architecture

*Four layers of fallback so your AI never stops working.*

---

## The Problem

AI APIs go down. Anthropic has outages. Rate limits hit. Free tiers disappear. If your entire workflow depends on one model from one provider, your business stops when that provider stops.

Most AI agent deployments ignore this. They pick Claude or GPT-4, hard-code it everywhere, and pray. We learned the hard way — through multiple outages, silent failures, and burned overnight budgets — that resilience isn't optional for production AI.

> Your 8 AM business briefing should arrive every morning. Even during an Anthropic outage. Even when you've burned through your daily budget on a complex project.

---

## Layer 1: Model Fallback Chains

Every agent has a chain of models, not a single model:

```
Alice:    Claude Opus (single — quality non-negotiable)
Builder:  Kimi K2.5 (free) → Claude Sonnet → Claude Opus
Mojo:     Kimi K2.5 (free) → Claude Sonnet → Claude Opus
CS:       Kimi K2.5 (free) → Claude Sonnet
Phoenix:  Claude Sonnet → Claude Opus
Scout:    Claude Sonnet → Claude Opus
Frank:    Claude Haiku (single — simple tasks, no fallback needed)
```

**The incident that spawned this:** Kimi K2.5 Free hit 429 rate limits during an overnight coding session. The builder stalled completely — no recovery path, no notification. Eight hours of potential work, lost. The next morning, fallback chains were added to every agent's gateway config.

**How it works:** When the primary model returns a 429 or 5xx, the gateway automatically routes to the next model in the chain. The agent doesn't know or care — it keeps working. Fallback triggers are logged for cost analysis.

**Design principle:** Alice stays on Opus with no fallback. Judgment, nuance, and strategic thinking can't be downgraded. Everything else can.

---

## Layer 2: Budget Cascade

AI that knows when to slow down:

```
Budget > 50%  → Full operations, proactive everything
Budget 30-50% → Conservative — reduce speculative checks
Budget 15-30% → Austerity — essentials only, alert the human
Budget &lt; 15%  → Emergency — urgent notification, minimal ops
```

**Frank the Budget Watchdog** monitors daily spend with a dedicated cron. When costs spike — maybe a complex analysis required more Opus calls than expected — Frank throttles non-essential crons to austerity mode. Essential briefings and monitoring keep running. The weekly Notion cleanup can wait.

**The math:** 47+ cron jobs across 9 agents could theoretically burn through a day's budget in hours if unchecked. The cascade prevents surprise overruns. You never wake up to a $50 overnight bill.

**Per-task caps:** During nightshift, each task gets a maximum of 15% of the nightly budget. This prevents one runaway task (a complex research deep-dive, a coding loop that hits edge cases) from eating the entire night's capacity.

---

## Layer 3: Self-Healing Infrastructure

The system doesn't just survive failures — it routes around them.

**Stall detection:** Every builder agent writes progress to a file in `/tmp/`. A watchdog checks file age. No progress for 20 minutes → kill the stalled agent, respawn with the same task file. The M\*-TASK.md pattern makes this possible — tasks are stateless, so respawning is cheap.

**Credential health monitoring:** A daily cron (`credential-health-check.sh`) tests every integration:
- Google Calendar OAuth
- Gmail via gog
- IMAP (work + alice accounts)
- GitHub SSH + CLI
- Notion API
- Obsidian vault git remote

If any credential is expired or broken, the cron reports failures only — no noise on success. This catches issues before they cascade through the escalation chain.

**The cascade failure that taught us this:** In W11, IMAP authentication failed on both email accounts. This blocked Annie's escalation → blocked Alice's verification → blocked Cordell's visibility into P1 payroll items aging 5+ days. A single credential failure cascaded through the entire system. Now credential health is checked proactively.

---

## Layer 4: Local Embedding Fallback

The most elegant resilience feature: when cloud search fails, local search takes over.

**Primary:** QMD hybrid search engine (BM25 + vector, 224MB SQLite, 1,457+ files indexed)
**Secondary:** Gemini free-tier vector search (cloud API)

**The incident:** During a heavy research session, Gemini free-tier rate limits killed Alice's ability to search her own knowledge base. Instead of just failing, Alice built a local embedding fallback using `node-llama-cpp` with the `embeddinggemma-300M` model. Runs entirely on-device. Zero API cost. Zero rate limit risk.

Now the local engine is primary and Gemini is secondary. Either way, search works. The "insurance" that your knowledge base is always queryable costs literally nothing extra to maintain.

---

## Model Routing: Right Model, Right Job, Right Cost

Resilience isn't just about surviving failures — it's about optimal routing during normal operations.

| Task Type | Model | Cost | Why |
|-----------|-------|------|-----|
| User conversations | Opus | Premium | Judgment, nuance, strategic thinking |
| Email drafting | Opus/Sonnet | Mid | Voice matching, relationship awareness |
| Email triage | Haiku | ~$0.02/run | Binary classification, pattern matching |
| Background crons (47+ jobs) | Haiku | ~$0.001/run | Simple operations, reliability > quality |
| Overnight coding | Kimi K2.5/Sonnet | ~$0.50/hr | Good code quality, 8x cheaper than Opus |
| CX auto-resolve | Python/rules | $0 | No AI needed — rule-based classification |
| Vector embeddings | Local (Ollama) | $0 | On-device, zero API cost |

**Result:** 8-12x cost reduction on background operations with zero quality degradation where it matters. The founder gets Opus-quality interaction. Email classification gets Haiku. Nobody's overpaying.

> Running everything on one premium model is like hiring a VP to answer phones. Route the work to the right level.

---

## The GLM-4.7-Free Incident: Why Free Models Are for Experiments, Not Production

In early February, ~50% of overnight crons were silently failing. The root cause: GLM-4.7-free returned 429 rate limit errors, but the crons showed "ok" status because the error wasn't propagated. The system appeared healthy. It wasn't.

**Discovery:** During the v2.0 "Mission Control" audit, Alice found that 19 of 28 crons had wrong model IDs — a bulk migration without validation. 5 were actively failing with "Unknown model" errors.

**Lesson:** Free models are for experimentation, not production automation. Now a hard rule in AGENTS.md: all production crons run on Haiku (cheap but reliable). Free-tier models are only used for background enrichment where failure is acceptable.

**The broader lesson:** After bulk operations, validate at least one actually runs before assuming all are fine. This became standard practice for all system-wide changes.

---

## Resilience Scorecard

After implementing all four layers:

| Failure Mode | Before | After |
|-------------|--------|-------|
| API outage | System stops | Automatic fallback, work continues |
| Rate limit | Agent stalls indefinitely | Routes to next model in chain |
| Budget overrun | Surprise bills | Cascade throttling, human alert |
| Agent stall | Discovered hours later | 20-min detection, auto-respawn |
| Credential expiry | Silent cascade failure | Daily proactive testing |
| Knowledge search down | No search capability | Local fallback, zero API cost |
| Wrong model ID | Silent failure for days | Validation on bulk changes |

> The system doesn't just work. It works despite failures. That's the difference between a demo and production.

---

*← Back to [README](../README) | Related: [Token Economics](token-economics) · [Infrastructure](../01-foundations/infrastructure)*
