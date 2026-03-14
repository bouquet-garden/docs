# Token Economics & Cost Control

*How intelligent model routing turns a $12/day expense into a $3/day operating system.*

---

## The Naive Approach

When most people start with AI agents, they pick one model and run everything on it. Claude Opus for email triage, Opus for background crons, Opus for monitoring, Opus for everything. That's $8-12/day — $240-360/month.

Not catastrophic, but not smart either. Email classification doesn't need Opus. A $0.001/run model classifies "spam or client email?" just as well as a $0.05/run model. Running everything on one model is a 50x overspend on commodity tasks.

> Running everything on one premium model is like hiring a VP to answer phones.

---

## The Routing Table

Every task gets assigned to the cheapest model that can handle it:

| Task Type | Model | Approximate Cost | Why This Model |
|-----------|-------|-------------------|----------------|
| User conversations | Opus | Premium | Judgment, nuance, strategic thinking |
| Email drafting | Opus/Sonnet | Mid-Premium | Voice matching, relationship awareness |
| Email classification | Haiku | ~$0.02/run | Binary decisions, pattern matching |
| Background crons (47+ jobs) | Haiku | ~$0.001/run | Simple monitoring, reliability matters more than quality |
| Overnight coding | Kimi K2.5 / Sonnet | ~$0.50/hour | Good code quality, 8x cheaper than Opus |
| CX auto-resolve | Python (no AI) | $0 | Rule-based classification, no model needed |
| Vector embeddings | Local (Ollama) | $0 | On-device, zero API cost |
| Fresh Eyes reviews | Sonnet | ~$0.01/review | Good quality/cost balance for QA |
| Sprint health checks | Haiku | ~$0.001/run | Formulaic checks, no reasoning needed |

**The math:** ~$3/day average across all operations. That's $90/month for a full-time AI Chief of Staff.

For context:
- A full-time ops person: $60-80K/year ($5-7K/month)
- A fractional COO: $2-5K/month
- Slack for a small team: ~$100/month
- Alice: $90/month — and she works 24/7

---

## Budget Watchdog (Frank)

Frank is a dedicated budget monitoring agent running on Haiku (the cheapest model — because your budget watchdog shouldn't be expensive).

**What Frank monitors:**
- Daily token spend across all agents
- Cost attribution by task type
- Budget cascade thresholds
- Anomaly detection (unexpected spikes)
- Weekly cost trend reports

**Budget cascade:**
```
Budget > 50%  → Full operations
Budget 30-50% → Conservative (reduce speculative checks)
Budget 15-30% → Austerity (essentials only)
Budget < 15%  → Emergency (alert Cordell, minimal ops)
```

**Why a dedicated agent:** Having Alice monitor her own budget is like asking the spender to audit the spending. Frank is an independent watchdog — the auditor watching the watchers.

---

## Delegation Economics: When to Use What

### The Decision Framework

```
Is this a judgment call? → Opus
Is this a creative task? → Opus or Sonnet
Is this classification? → Haiku
Is this rule-based? → No AI (Python script)
Is it background/monitoring? → Haiku
Is it overnight coding? → Cheapest model that produces working code
```

### The Quality-Cost Frontier

| Model | Strength | Weakness | Use When |
|-------|----------|----------|----------|
| **Opus** | Nuance, judgment, complex reasoning | Expensive, slower | User-facing, strategic decisions, complex analysis |
| **Sonnet** | Good balance, reliable | Not as sharp as Opus on edge cases | Email drafting, code review, research synthesis |
| **Haiku** | Fast, cheap, reliable | Limited reasoning | Monitoring, classification, simple operations |
| **Kimi K2.5** | Free/cheap, good code generation | Rate limits on free tier | Overnight coding, batch operations |
| **No AI** | Free, deterministic, fast | No flexibility | Rule-based classification, API calls, file operations |

### The Real Cost Breakdown (Typical Day)

| Activity | Count | Model | Est. Cost |
|----------|-------|-------|-----------|
| Morning briefing | 1 | Haiku | $0.01 |
| User conversations | 5-10 | Opus | $1.50 |
| Email triage | 2 | Haiku | $0.04 |
| Cron jobs | 20+ | Haiku | $0.05 |
| Meeting processing | 1 | Sonnet | $0.10 |
| Research/analysis | 1 | Sonnet | $0.20 |
| Overnight shift | 6 tasks | Mixed | $1.50 |
| **Total** | | | **~$3.40** |

---

## The Free Tier Lesson

In January, many crons ran on free-tier models (GLM-4.7-Free). Cost: $0. Reliability: terrible.

~50% of overnight crons were silently failing. GLM returned 429 rate limits but crons reported "ok" because errors weren't propagated. The system appeared healthy. It wasn't.

**The rule that emerged:** Free models are for experimentation, not production automation. Background enrichment that can fail gracefully? Free tier is fine. Morning briefing that must arrive? Haiku minimum.

**The sweet spot:** Haiku at ~$0.001/run is cheap enough to be considered "basically free" but reliable enough for production. That's the production floor.

---

## Cost Comparison: Alice vs. Alternatives

| Solution | Monthly Cost | Coverage | Context |
|----------|-------------|----------|---------|
| **Alice** | ~$90 | 24/7, all domains | Full business context, compounds over time |
| Full-time EA | $4-6K | Business hours, one person | Deep but narrow, turnover risk |
| Fractional COO | $2-5K | 10-20 hrs/week | Strategic but limited availability |
| Serif.ai | $49/mo | Email only | No business context, no memory |
| Zapier automations | $100+ | Trigger-action only | No judgment, no cross-domain reasoning |
| VA service | $500-2K | Business hours, variable | Quality variance, training overhead |

**The key differentiator:** Alice has cross-domain context that no human or tool can match. She reads email, Notion, Slack, calendar, meeting transcripts, and the knowledge vault simultaneously. No individual person — at any price — holds that much operational context.

---

## Scaling Economics

As usage grows, two forces compete:

**Costs that scale linearly:**
- More conversations → more Opus tokens
- More crons → more Haiku tokens
- More agents → more model calls

**Costs that scale sub-linearly (the compound advantage):**
- Solutions library → fewer new solutions needed
- Rules → fewer errors → fewer recovery costs
- Memory → less context needed per query (the answer is already documented)
- Scripts → automation replaces AI calls (CX triage costs $0/run permanently)

The compound learning system means the marginal cost of the Nth task is lower than the first task. Solutions get reused. Patterns get encoded. Scripts replace model calls. The system gets cheaper to operate over time.

---

*← Back to [README](../README.md) | Related: [Model Resilience](model-resilience.md) · [The Nightshift](nightshift.md)*
