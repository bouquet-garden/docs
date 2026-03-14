---
title: "Case Study: 565 CX Conversations Resolved in 3 Minutes"
---

# Case Study: 565 CX Conversations Resolved in 3 Minutes

*How reverse-engineering a REST API eliminated a full-time support role at $0 AI cost.*

---

## The Problem

Superculture's Chatwoot customer support inbox (hosted on Commslayer) had accumulated 565 open conversations. The inbox was unusable — genuine customer issues were buried under hundreds of vendor notifications, system alerts, hiring messages, and spam.

Opening the inbox felt like drowning. Every conversation looked urgent. None were prioritized. The visual chaos made it impossible to find the 5% of conversations that actually needed human attention.

The math was simple and painful:
- A full-time CS rep to manage this: **$45K/year**
- A fractional support person: **$500-1,000/month**
- The conversations were accumulating faster than any human could triage them
- The 95% that didn't need attention were stealing focus from the 5% that did

## The Approach

### Step 1: Reverse-Engineer the API

Chatwoot (via Commslayer) doesn't publish a comprehensive public API for bulk operations. Most people would stop here — "there's no API, so we can't automate it."

Alice opened browser dev tools, performed actions in the web app, and watched the Network tab. This is one of the most valuable operational patterns in the entire system: **any web app has an API — they just don't always publish it.**

**What she found:**
- REST API base: `https://app.commslayer.com/api/v1/accounts/{id}/conversations`
- Authentication: session cookies (`access-token`, `token-type`, `client`, `uid`)
- Pagination: standard page parameter for scanning the full inbox
- Status toggle: POST to `/{conv_id}/toggle_status` with `{"status": "resolved"}`
- Conversation metadata includes labels, timestamps, and assignment status

The entire API was mapped in about 20 minutes of browser dev tools observation. No documentation needed. No vendor support tickets. Just: watch the network tab, reproduce the pattern.

### Step 2: Design Classification Logic

Here's the critical insight: **this didn't need AI at all.** The conversations were already labeled in Chatwoot. The classification was pure label-based logic:

| Label | Action | Rationale |
|-------|--------|-----------|
| `notification` | Auto-resolve | System alerts — order confirmations, shipping updates, platform notifications |
| `vendor` | Auto-resolve | Vendor communications handled in other channels (email, WhatsApp) |
| `hiring` | Auto-resolve (optional flag) | Hiring pipeline managed in Notion, not Chatwoot |
| `customer-support` | **Keep open** | Genuine customer issues requiring human attention |
| *(unlabeled)* | **Keep open** | Unclassified = potentially important, needs human review |

No sentiment analysis. No intent detection. No NLP. Just: "Is this labeled 'notification'? Resolve it." The right tool for the job was a Python script, not a language model.

### Step 3: Build the Script

`commslayer-triage.py` — 80 lines of Python. No AI framework. No LangChain. No vector database. Just `urllib.request` and `concurrent.futures`.

**Key design decisions:**

1. **Dry-run mode by default.** The script previews everything before touching anything. The first run showed exactly what would be resolved — no surprises, no "oops, resolved a real customer issue." This is a non-negotiable pattern for any automation that modifies production data.

2. **Concurrent resolution for speed.** Using `concurrent.futures.ThreadPoolExecutor` to resolve multiple conversations simultaneously. Sequential resolution of 565 conversations would take 10+ minutes; concurrent execution brought it under 3.

3. **Page-by-page scanning.** The inbox is paginated — the script handles any inbox size by iterating through all pages until exhausted. Works for 565 conversations or 5,000.

4. **Statistics tracking.** Every run outputs a breakdown: how many resolved by label, how many kept open, how many errors. Full audit trail.

5. **Auth from file.** Session cookies are loaded from a JSON file (`scripts/data/commslayer-auth.json`), not hardcoded. When cookies expire (user logs in again), update the file — no code changes needed.

```bash
# Preview what would be resolved (safe — changes nothing)
python3 scripts/commslayer-triage.py

# Actually resolve (after reviewing the preview)
python3 scripts/commslayer-triage.py --execute

# Include hiring conversations in the resolution
python3 scripts/commslayer-triage.py --execute --resolve-hiring
```

### Step 4: Execute

```bash
python3 scripts/commslayer-triage.py --execute
```

565 conversations resolved in 3 minutes. The inbox went from chaos to a clean, manageable queue of genuine customer issues.

### Step 5: Ongoing Maintenance

The script isn't a one-time fix. It runs on a schedule via cron, keeping the inbox clean as new conversations accumulate. The notification/vendor/hiring pipeline never stops generating inbox noise — the script never stops cleaning it up.

## The Results

| Metric | Value |
|--------|-------|
| Conversations resolved | 565 |
| Time to resolve | 3 minutes |
| AI cost | **$0** (no LLM calls) |
| Script size | 80 lines of Python |
| Script development time | ~1 hour |
| Annual CS rep cost avoided | $45,000 |
| Libraries used | `urllib.request`, `concurrent.futures` (stdlib only) |
| Script reusability | Runs on any schedule for ongoing maintenance |

## The Technical Pattern: Browser Dev Tools → API → Automation

This case study demonstrates a pattern that's applicable to any web app without a public API:

1. **Open browser dev tools** (Network tab)
2. **Perform the action manually** in the web UI
3. **Watch the network request** — note the URL, method, headers, payload
4. **Reproduce in code** — usually 10-20 lines of Python
5. **Build the automation** around the discovered endpoint

This pattern works on Chatwoot, Accio (Alibaba), internal admin panels, SaaS tools with limited APIs — essentially any web application that has a UI. If a human can click a button to do it, an agent can find and call the API behind that button.

Alice uses this pattern across multiple integrations. It's one of the highest-leverage technical capabilities in the system.

## The Lessons

**Not everything needs AI.** The industry obsession with throwing LLMs at every problem is expensive and often wrong. The 565 conversations were already classified by existing labels. The right tool was 80 lines of Python and zero API calls.

**Reverse-engineering APIs is a superpower.** Every web app has an API — they just don't always publish it. Browser dev tools + network tab + 20 minutes of observation = automation capability for any platform. This is one of the most reusable patterns in Alice's toolkit.

**Dry-run modes are non-negotiable.** The script defaults to preview mode. The first run showed exactly what would be resolved before touching anything. This prevented the catastrophic failure mode: accidentally resolving a real customer issue. For any automation that modifies production data, dry-run-by-default is a design principle, not a nice-to-have.

**Simple > clever.** No vector embeddings. No fine-tuned classifier. No RAG pipeline. Just label-based routing and an HTTP client. The solution that works in 80 lines of stdlib Python is better than the solution that works in 800 lines with 12 dependencies.

> A full-time CS rep costs $45K/year. This script cost $0 to run, ~$1 of compute to develop, and resolves conversations in perpetuity. The math isn't close. And the pattern — reverse-engineer any web app's API from the browser network tab — applies to every tool in your stack that doesn't have a public API.

---

*← Back to [README](../README) | Section: [Case Studies](./)*
