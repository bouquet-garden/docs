---
title: "Email Triage Pipeline Playbook"
---

# Email Triage Pipeline Playbook

*From inbox chaos to classified, tracked, and draft-ready — twice daily.*

---

## Overview

The email triage pipeline transforms a raw inbox into a structured, prioritized, action-ready state. It runs twice daily (morning and evening), with fast polls every 30 minutes for urgent items.

This system replaced manual email triage and outperforms Serif.ai by leveraging cross-surface context from Notion, Slack, Granola, Obsidian, and relationship history.

---

## The Pipeline

### Phase 1: Inbox Assessment

```bash
# Pull recent emails (work account)
himalaya envelope list -a work --page-size 30

# Personal account (if needed)
himalaya envelope list -a personal --page-size 20

# Search specific patterns
himalaya envelope list -a work -s 10 'from kevin'
himalaya envelope list -a work -s 10 'after 2026-03-01'
```

**Output:** List of unread emails (marked with `*`) with sender, subject, date.

### Phase 2: Classification

Each email gets assigned exactly one priority:

| Priority | Definition | Action |
|----------|-----------|--------|
| P0 Urgent | Client crises, payment issues, team blockers | Alert immediately, draft response |
| P1 Important | Client questions, vendor comms, team updates | Draft response in next triage |
| P2 FYI | Newsletters, notifications, non-urgent updates | Read, label, archive |
| P3 Archive | Spam, irrelevant forwards, old threads | Archive immediately |

### Phase 3: Thread Ownership

Every email thread gets classified:

```json
{
  "thread-id-123": {
    "owner": "us",          // Ball is with us — we owe a response
    "lastActivity": "2026-03-08T14:30:00Z",
    "staleHours": 18,
    "escalation": "normal"  // &lt; 24h for our ball
  }
}
```

**Ownership signals:**
- "Looking forward to hearing from you" → Ball with us
- "I'll get back to you on that" → Ball with them
- "Please review and let me know" → Ball with us
- "Sending this over for your files" → No action needed

**Staleness rules:**
| Ownership | Hours | Status |
|-----------|-------|--------|
| Our ball | &lt; 24h | Normal |
| Our ball | 24-48h | Warning |
| Our ball | > 48h | Urgent escalation |
| Their ball | &lt; 48h | Normal monitoring |
| Their ball | > 48h | Suggest follow-up nudge |

### Phase 4: Draft Preparation

For P0 and P1 emails, Alice drafts responses with context:

```bash
echo 'From: work@email.com
To: kevin@portless.com
Subject: Re: Order #4521 Status

Hi Kevin,

[Response body — informed by Notion project status, Slack conversations, 
meeting transcript references, and relationship history]

Best,
Cordell

<!-- Alice Notes (stripped before sending):
- Related Notion task: "Portless merchant agreement review" (P2, due next week)
- Discussed in Feb 14 coaching call — timeline has 3-day buffer
- Shyla flagged packaging concern on Slack 2 days ago
- Kevin responds within 24h typically, reliable vendor
-->' | himalaya message save -a work -f "[Gmail]/Drafts"
```

**The Alice Notes pattern:** Red HTML comment blocks with context that help Cordell review the draft. These get stripped before sending. They include:
- Related Notion tasks and status
- Relevant Slack conversations
- Meeting transcript references
- Relationship context (response patterns, reliability)

### Phase 5: Gmail Label Sync

| Label | Meaning | Applied When |
|-------|---------|-------------|
| `1: Needs Response` | Ball with us, action required | Thread ownership = us |
| `3: Waiting for Reply` | Ball with them, monitor | Thread ownership = them |

### Phase 6: Notification

```
📬 Email Triage Complete

P0 (Urgent): 1 — Kevin at Portless (order timeline question)
P1 (Important): 3 — Felix re: tech packs, Shyla re: ShipBob, new vendor inquiry
P2 (FYI): 7 — newsletters, notifications
P3 (Archived): 12

Drafts ready: 4 (in Gmail Drafts folder)
Stale threads (our ball): 2 (>24h)

[Review Drafts] [Mark All Read]
```

---

## The Outbound Gate Protocol

**THE MOST IMPORTANT RULE IN THE EMAIL SYSTEM.**

Born from the February 13 incident: Cordell asked Alice to "draft" email responses. Alice interpreted this as authorization to send live emails to external business contacts.

### The Rule

| Command | Action | Safe? |
|---------|--------|-------|
| "draft" | `himalaya message save -f "[Gmail]/Drafts"` | ✅ Always safe |
| "prepare" | Save to drafts | ✅ Safe |
| "write" | Save to drafts | ✅ Safe |
| "finalize" | Save to drafts | ✅ Safe |
| "approve" | Save to drafts | ✅ Safe |
| **"send"** | `himalaya template send` | ⚠️ Only with this exact word |

**Only the word "send" authorizes actual email delivery.** All other verbs default to saving drafts.

### Safety Commands

```bash
# SAFE — saves to drafts folder, never sends
echo '...' | himalaya message save -a work -f "[Gmail]/Drafts"

# DANGEROUS — actually sends the email (requires explicit "send" instruction)
echo '...' | himalaya template send -a work
```

---

## Cross-Surface Context

What makes Alice's email triage better than Serif.ai or generic email AI:

| Context Source | What It Provides | Example |
|---------------|-----------------|---------|
| **Notion** | Project status, task state | "This order is tracked in sprint, P2, due next week" |
| **Slack** | Team discussions | "Shyla mentioned packaging concerns 2 days ago" |
| **Granola** | Meeting transcripts | "Discussed in Feb 14 coaching call" |
| **Obsidian** | SOPs, historical context | "Our vendor evaluation checklist says..." |
| **World state** | Unified business context | "Kevin's response time is typically &lt;24h" |
| **Memory** | Relationship history | "We've worked with this vendor since January" |

A response drafted with this context is fundamentally different from a response drafted with only the email thread. The recipient gets a reply that reflects full business awareness, not just inbox awareness.

---

## Fast Poll (Urgent Detection)

Every 30 minutes during business hours:

```
1. Check for new unread emails
2. Quick-classify: is anything P0?
3. If P0 detected: immediate Telegram alert
4. If no P0: wait for next scheduled triage
```

**Vendor Hours Poll:** Separate check during 4-5 PM PST (8-9 AM China time) specifically for factory and supplier responses. Catches vendor replies when they're most likely.

---

## Setting Up Email Triage

### Prerequisites
- Himalaya configured with work account (`~/.config/himalaya/config.toml`)
- Gmail labels created: `1: Needs Response`, `3: Waiting for Reply`
- World state cache active (Notion, Slack data refreshed)
- Telegram notifications working

### Cron Setup
```
Morning triage: 10 AM PT
Evening triage: 8 PM PT
Fast poll: Every 30 min during 8 AM - 8 PM PT
Vendor hours: 4-5 PM PT (China morning)
Follow-up tracker: Daily at 11 AM
```

### Tuning
- **Classification accuracy:** Review first week's classifications. Adjust patterns for false positives.
- **Thread ownership detection:** Add domain-specific ownership signals.
- **Draft quality:** Review drafts for tone and accuracy. Update relationship context in memory.
- **Staleness thresholds:** Adjust 24h/48h thresholds based on business pace.

---

## Common Issues

**Himalaya auth expiry:** OAuth tokens expire periodically. Re-authenticate via `himalaya account configure work`. Symptoms: "authentication failed" errors in triage cron.

**Label sync failures:** Gmail API rate limits can prevent label application. Non-critical — labels are convenience, not system-critical.

**Draft context staleness:** If world state cache hasn't refreshed, drafts may reference outdated Notion task status. Ensure cache refresh cron runs before triage.

---

*The email pipeline handles 200+ emails/month across work and personal accounts. Automated triage saves approximately 30-60 minutes/day of manual classification. Draft quality (with cross-surface context) saves an additional 15-30 minutes/day of response writing.*

*See also: [Skills & Integrations](../tier1/skills-integrations.md) for himalaya details, [Operational Results](../tier3/operational-results.md) for email metrics.*

---

*← Back to [README](../README.md) | Section: [Playbooks](./)*
