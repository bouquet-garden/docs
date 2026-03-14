# Cron as Proactivity

**47 crons = giving AI the ability to initiate.**

Most AI is reactive. You prompt, it responds. You ask, it answers. You push a button, it does a thing.

That's a tool. Not a team member.

A team member doesn't wait for you to ask "any new emails?" A team member checks email, notices something important, and tells you. That's initiative. That's proactivity. And in AI agent systems, proactivity runs on cron jobs.

---

## The Thesis

**Agent infrastructure matters more than model quality.**

GPT-4, Claude, Gemini — they're all smart enough to draft an email, summarize a meeting, or write code. The difference between "interesting demo" and "production AI team" isn't which model you use. It's whether the model runs when you're not looking.

A brilliant model that only activates when you type a prompt is a very fast employee who sits in the break room until you physically walk over and assign them something.

A competent model running on 47 scheduled triggers is a team that works 24/7, monitoring your business surfaces, flagging problems, executing routine work — whether or not you remembered to ask.

> *Tweetable: "The difference between AI tools and AI agents isn't intelligence. It's initiative. Tools wait for you to ask. Agents check before you think to."*

---

## What 47 Crons Actually Look Like

### Communications (Annie — 7 crons)
| Cron | Frequency | What It Does |
|------|-----------|-------------|
| Email scan | Every 30 min | Check inbox, classify, triage, draft responses for review |
| Slack monitor | Every 30 min | Scan channels, detect mentions, flag action items |
| Vendor reply watch | Every 30 min | Monitor Accio/Alibaba for supplier responses |
| WhatsApp check | Hourly | Surface business messages from noise |
| Delegation detector | Every 30 min | Identify emails that delegate work to Cordell |
| Escalation scanner | Every 2 hours | Flag emails that need immediate attention |
| Weekend digest | Saturday 9 AM | Consolidated communications from the week |

### Operations (Frank — 4 crons)
| Cron | Frequency | What It Does |
|------|-----------|-------------|
| Budget watchdog | Every 4 hours | Monitor token spend, trigger austerity if needed |
| Weekly ops review | Wednesday 9 AM | Per-agent metrics, cron health, learning audit |
| Cross-signal detection | 2x daily | Find entities mentioned across multiple surfaces |
| System health | Every 6 hours | Verify all agents, crons, and integrations are alive |

### Coordination (Alice — 15 crons)
| Cron | Frequency | What It Does |
|------|-----------|-------------|
| Morning briefing | 8:00 AM weekdays | Consolidated intelligence from all surfaces |
| Sprint standup | 9:00 AM weekdays | Sprint status + priorities for the day |
| Hourly memory | Every hour | Structured summary for context persistence |
| CONTEXT.md sync | Every 2 hours | Update compaction-proof state file |
| Notion sync | Hourly | Reconcile sprint board with local state |
| Meeting processor | Post-meeting | Transcript → structured Notion tasks |
| EOD review | 6:00 PM weekdays | Day summary + overnight task scoring |
| Overnight task scoring | 10:00 PM | Evaluate and prioritize nightshift work |
| Nightshift executor | 10:30 PM - 6:00 AM | Autonomous work across 5 domains |
| Morning digest compile | 7:00 AM | Package overnight results for delivery |
| Calendar scanner | Every 4 hours | Detect conflicts, prep for upcoming meetings |
| Feedback aggregator | Daily | Consolidate operator feedback into learning |
| Weekly synthesis | Friday 5 PM | Review all learnings, consolidate patterns |
| Rule enforcement audit | Weekly | Verify incident-derived rules are being followed |
| Stale task detector | Daily | Flag tasks with no progress for >3 days |

*(Plus additional domain-specific crons for vendor monitoring, content scheduling, and system maintenance.)*

---

## Coverage, Not Conversation

Here's the reframe that changes how you think about AI:

### The old model: conversation
You open ChatGPT. You type a question. You get an answer. You close the tab. The AI exists only while you're talking to it.

**Coverage time per day: ~30 minutes** (however long you chat).

### The new model: coverage
47 crons run throughout the day. Each one checks a surface, evaluates conditions, takes action or surfaces information. The AI is working whether you're looking at it or not.

**Coverage time per day: 24 hours.** Not because any single cron runs for 24 hours, but because they're staggered across the full day. Something is always watching.

### The math
- 47 crons × average 2 runs/day = ~94 automated checks per day
- Each check takes 10-60 seconds
- Total compute time: ~30-60 minutes (spread across 24 hours)
- Cost: ~$0.50/day (Haiku model for most crons)
- Coverage: every email, every Slack message, every vendor reply, every calendar conflict, every sprint status change — detected within 30 minutes of occurrence

For $0.50/day, you get a team that checks everything, 24/7. A human doing the same thing would spend 4-6 hours/day just on the checking. Not doing anything about what they find — just checking.

> *Tweetable: "47 cron jobs. $0.50/day. 24/7 coverage of every surface in my business. Not AI conversation — AI coverage."*

---

## The Proactivity Spectrum

Not all crons are created equal. They exist on a spectrum from passive monitoring to active intervention:

### Level 1: Monitor and report
"Check email. If anything new, summarize and surface."

The cron reads a surface and tells you what it found. You decide what to do. This is the simplest form of proactivity — the AI looks so you don't have to, but all decisions remain human.

### Level 2: Monitor, classify, and recommend
"Check email. Classify by urgency. Recommend responses for the urgent ones."

The cron adds judgment. It doesn't just tell you "5 new emails" — it tells you "2 urgent (vendor deadline + client escalation), 2 routine (newsletter + receipt), 1 spam." And for the urgent ones, it drafts a response for your review.

### Level 3: Monitor, classify, and act (within boundaries)
"Check email. If it's a newsletter, archive. If it's a vendor reply, update the comparison matrix. If it's a client escalation, draft response and notify me."

The cron handles routine actions autonomously and escalates exceptions. This is where the internal/external delegation rule applies: internal actions (archive, update matrix) are autonomous. External actions (send reply to client) require approval.

### Level 4: Autonomous operation with oversight
"Run overnight. Evaluate all pending tasks. Execute the top-scored ones across 5 domains. Budget-cap at 60%. Deliver digest at 7 AM."

The nightshift protocol. Full autonomous operation within well-defined guardrails. The operator sleeps. The AI works. The morning digest is the accountability layer.

### Alice's distribution
| Level | Cron Count | Description |
|-------|-----------|-------------|
| Level 1 | ~10 | Pure monitoring (system health, Slack scan) |
| Level 2 | ~20 | Monitor + classify (email triage, vendor tracking) |
| Level 3 | ~12 | Monitor + classify + act (sprint updates, memory sync) |
| Level 4 | ~5 | Autonomous operation (nightshift, weekly synthesis) |

Most crons are Level 2 — smart monitoring. The progression from Level 1 to Level 4 is the trust calibration described in `delegation-philosophy.md`.

---

## Why Crons Beat Event-Driven Architecture

The objection we hear: "Why scheduled polling instead of event-driven triggers? Webhooks are more efficient."

### The practical answer
1. **Not everything has webhooks.** Alibaba/Accio doesn't push notifications to your agent. Gmail API has push notifications, but they're unreliable. Calendar APIs have sync tokens, but they expire. Crons work with everything, regardless of API sophistication.

2. **Batch processing is natural.** A cron that runs every 30 minutes processes all accumulated changes in one pass. An event-driven system processes each change individually, with all the cold-start and context-loading overhead that implies.

3. **Rate limiting is built in.** A cron runs at most once per interval. An event-driven system can fire 100 times in a second if 100 emails arrive simultaneously. The cron naturally rate-limits; the event system needs explicit throttling.

4. **Debugging is trivial.** "What happened at 9:30 AM?" Check the 9:30 cron log. Event-driven systems scatter their execution across time in ways that are harder to trace.

### The philosophical answer
Crons align with how operators think. An operator doesn't think "when this event fires, execute this handler." An operator thinks "every morning, check these 5 things." Cron jobs are the operator's mental model made executable.

> *Tweetable: "We chose crons over webhooks. Not because they're technically superior — because they match how operators think. 'Check this every 30 minutes' is a management instruction, not a systems architecture."*

---

## The Compound Effect of Coverage

Individual crons are boring. "Check email" isn't impressive. "Scan Slack" isn't impressive.

47 crons running together, sharing context through a unified memory system? That's something else.

### The cross-signal example
- **9:00 AM:** Email cron detects vendor message about order #4521
- **9:15 AM:** Memory system connects it to meeting transcript from last week
- **9:30 AM:** Calendar cron confirms deadline is next Friday
- **9:35 AM:** Slack cron finds teammate's packaging concern from two days ago
- **9:40 AM:** Alice surfaces a consolidated briefing: "Vendor confirmed shipment for #4521. Lands 3 days before deadline per your Feb 14 call. Note: teammate flagged packaging question — confirm specs before it ships."

No single cron produced that intelligence. The morning briefing synthesized signals from 4 different crons, cross-referenced against memory. This is **contextual proactivity** — not just checking surfaces, but connecting what's found across them.

This can't be built with tools. It can't be built with conversation. It requires coverage — continuous, systematic monitoring of multiple surfaces with shared context. That's what 47 crons provide.

---

## Building Your Cron Architecture

### Start with 3 crons
1. **Morning briefing** (daily, 8 AM): Check email + calendar + one other surface. Summarize.
2. **Email monitor** (every 30 min): Classify incoming email. Surface urgencies.
3. **EOD review** (daily, 6 PM): Summarize the day. Flag unfinished work.

That's it. Three crons. You'll feel the difference within a week.

### Add incrementally
Each new cron should solve a specific problem you're currently solving manually:
- "I keep forgetting to check Slack" → Slack monitor cron
- "Vendor replies sit for hours" → Vendor watch cron
- "I lose context between days" → Memory sync cron

### The rule of thumb
**If you check it manually more than once a day, it should be a cron.**

Not everything needs to be. Some things are genuinely ad-hoc. But the routine checks — the stuff you do out of habit, not out of creative thought — that's cron territory.

### Model routing matters
Don't run 47 crons on your most expensive model. Most monitoring tasks need Haiku-level intelligence. Save Opus for the synthesis and judgment crons (morning briefing, overnight task scoring, cross-signal detection).

---

## The Future: From Coverage to Anticipation

Right now, crons are reactive-at-a-schedule — they check what's already happened and surface it. The next evolution is anticipation:

- "Based on vendor response patterns, this supplier probably won't reply until Thursday. Start engaging the backup now."
- "Based on sprint velocity, you'll miss next Friday's deadline by 2 days. Recommend renegotiating or cutting scope."
- "Based on email patterns, your client is about to ask for a status update. Draft one proactively."

This isn't science fiction. It's the same cron architecture with compound learning feeding prediction into the monitoring cycle. The agent doesn't just know what happened — it knows what's *likely* to happen, and acts before you ask.

> *Tweetable: "Most AI agents are reactive: you prompt, they respond. 47 crons make AI proactive: it monitors, detects, surfaces, and acts. Coverage, not conversation."*

---

*The infrastructure that lets AI initiate isn't glamorous. It's cron jobs. Scheduled tasks. Timer-based polling. The most boring technology in computing, applied to the most exciting capability in AI: giving models the ability to act first.*
