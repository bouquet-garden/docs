---
title: "The Failure Catalog"
---

# The Failure Catalog

**Builders trust builders who show the ugly parts.**

Every polished case study hides a graveyard of bad decisions. This is our graveyard, fully exhumed. Not because we're proud of these failures, but because they're the reason the system works now. Each one became a permanent rule — scar tissue that makes the organism stronger.

If you're evaluating Alice as a product, as a framework, or as a philosophy — this document tells you more than any demo ever could.

---

## The 8-Agent Trap

### What happened
We planned 8 agents: Alice, Dexter, Mojo, Phoenix, Scout, Cortex, Comms, and CS. It looked great on paper. Clean separation of concerns. Dedicated roles. Professional-sounding titles.

The math was brutal: 8 agents = 28 possible coordination pairs. Every pair is a potential miscommunication. Every handoff is a latency spike. Every agent needs its own context window, its own memory, its own cron budget.

We were building a bureaucracy, not a team.

### What we actually needed
4 core agents. Alice (coordination), Dexter (code), Mojo (design), Phoenix (infrastructure). Three more on-demand for specific cron domains: Scout, Frank, Annie. They wake up, do their job, go back to sleep.

Cortex got merged into Phoenix. Comms got merged into Alice. The coordination overhead dropped from 28 pairs to 6.

### The lesson
**More agents ≠ more capability.** Amazon's Two-Pizza Teams principle applies to AI teams too. If your agent architecture requires an org chart, you've already lost.

> *Tweetable: "We built 8 AI agents and discovered we'd accidentally invented middle management."*

### The rule it became
Agent team review now starts with: "Can the existing team handle this, or does the problem genuinely require a new agent?" The answer is almost always: the existing team can handle it.

---

## alice.db: The Database That Solved Nothing

### What happened
Week 2. We built `alice.db` with 8 SQLite tables. Task candidates, thread ownership, budget state, cross-signals, learnings, agent state, metrics, and a kitchen sink table we called "context."

It was architecturally pristine. Normalized. Indexed. Had a shell helper script. Every agent would read and write to it. Coordination through data, not conversation.

### Why it failed
Nobody needed it yet.

We had 4 agents doing simple tasks. The "coordination overhead" we were solving for was theoretical. The agents were spending more tokens parsing SQL results than they would have spent just... talking to each other through files.

The database was a solution looking for a problem. We'd read about Convex-backed shared databases in Bhanu's Mission Control thread (3.7M views) and cargo-culted the pattern without his scale.

### What replaced it
Flat files. `WORKING.md` per agent. Daily notes in markdown. JSON for state that needed structure. Simple, readable, zero overhead.

When we actually need a shared database — when agents are doing enough concurrent work that file-based coordination creates race conditions — we'll build one. But we'll build it because we *need* it, not because it looks good in an architecture diagram.

### The lesson
**Don't build infrastructure for problems you don't have yet.** Over-engineering is the most comfortable form of procrastination.

> *Tweetable: "We built a database with 8 tables for a system that needed 3 text files."*

---

## Layer 3 Task Inference: The AI That Predicted Work (Badly)

### What happened
The idea was seductive: have the agent watch your communication streams and *infer* tasks before you assign them. Email from a vendor about a deadline? Auto-create a task. Slack message about a bug? Task. Calendar conflict? Task with resolution options.

We called it Layer 3. Layer 1 was explicit tasks (you tell the agent what to do). Layer 2 was scheduled tasks (crons). Layer 3 was inferred tasks — the agent figures out what needs doing from ambient signals.

### Why it failed
The inference was either obvious (yes, I know I have a meeting tomorrow) or wrong (no, that email doesn't require a task — it's an FYI). The signal-to-noise ratio was terrible.

Worse: the system created a `task-candidates.json` file and a supervision cron to review it. Both ran daily. Both consumed tokens. Both produced nothing actionable. We found them months later during a cron audit — two no-op crons faithfully burning budget on an empty JSON file.

### What replaced it
Nothing. And that's the point. Not every problem needs a Layer 3. Sometimes Layer 1 (you tell the agent) and Layer 2 (scheduled monitoring) are sufficient. The agent doesn't need to *predict* your work — it needs to *execute* it well.

### The lesson
**Ambient intelligence sounds magical. Executing assigned work reliably is actually magical.** Don't build prediction when execution is still unreliable.

> *Tweetable: "We killed our AI task prediction engine. Turns out 'do what I tell you, well' is the harder and more valuable problem."*

---

## The Mavis Incident: Wrong Vendor, Right Message

### What happened
January 27, 2026. We were sourcing woven labels from multiple Alibaba/Accio suppliers. The agent needed to send a follow-up message to Bella, our preferred vendor.

She sent it to Mavis instead.

The message itself was perfect — correct specs, correct terminology, professional tone. It just went to the wrong person. The agent didn't verify which chat window was active before sending.

### The damage
Moderate embarrassment. Mavis got a message clearly intended for another supplier, revealing our comparison process. Not catastrophic, but unprofessional.

### What changed
Verification before action on external systems became mandatory. The agent now confirms recipient identity before sending any outbound communication. It's a 2-second check that prevents a class of errors that could range from embarrassing to legally problematic.

### The lesson
**Correct content + wrong recipient = wrong.** Verification isn't overhead. It's the difference between professional and amateur.

> *Tweetable: "Our AI agent sent a perfect message to the wrong vendor. That's when we learned: delivery accuracy matters more than content quality."*

---

## The Backup Vendor Purge

### What happened
Same sourcing project. We selected our primary woven label supplier. The agent, being helpfully efficient, sent polite decline messages to all backup vendors. Thanked them for their time, closed the conversations.

Seemed logical. We'd made our choice. Why keep people waiting?

### Why it was a disaster
Two weeks later, the primary supplier had a production delay. We needed to re-engage backup vendors. But they'd all received "thanks but no thanks" messages. Re-engaging cold after a formal decline is awkward, slow, and damages trust.

We lost a week. The delay cascaded into our production timeline.

### What changed
Hard rule: **never close out optionality without explicit human approval.** The agent can *recommend* declining backups. She cannot *execute* it autonomously.

This rule extends beyond vendor management. It applies to any decision that eliminates future options: canceling subscriptions, archiving projects, declining opportunities. Reversible decisions can be autonomous. Irreversible ones require approval.

### The lesson
**Efficiency and optionality are in tension.** An agent optimized purely for efficiency will close doors you might need open. The human's job is to manage optionality; the agent's job is to manage execution.

> *Tweetable: "Our AI agent helpfully declined all our backup vendors. Then our primary supplier missed their deadline. Now there's a rule: never close doors the human didn't explicitly shut."*

---

## 19 of 28 Crons: The Silent Failure Cascade

### What happened
February 9, 2026. During a system self-review, we discovered that 19 of our 28 cron jobs had the wrong model ID. The format was `claude-haiku-3-5` when it should have been `claude-3-5-haiku`.

5 of those 19 were actively failing — returning "Unknown model" errors on every run. The other 14 happened to work because they had cached sessions from before the ID was validated.

### How it happened
Bulk migration. We moved all crons from a free-tier model (GLM-4.7-free, which was hitting rate limits) to Haiku. Copy-pasted the wrong model ID format across all 19 crons. Didn't test a single one before marking the migration complete.

### The cascading failure
The 5 failing crons included monitoring tasks. Which meant:
- Vendor reply monitoring was down (missed supplier messages)
- Budget watchdog was down (no spend alerts)
- Two system health checks were down (no one watching the watchers)

We were flying blind and didn't know it.

### What changed
Two rules:
1. **After any bulk operation, validate at least one instance actually works before moving on.** Not "check the config looks right." Run it. Confirm output.
2. **Copy-paste model IDs from a verified reference, never from memory.** We added a model ID reference section to TOOLS.md.

### The lesson
**"It's a simple find-and-replace" is the prelude to every bulk failure.** The confidence that makes you skip validation is exactly the confidence that gets you burned.

> *Tweetable: "19 of 28 crons had the wrong model ID. 5 were failing silently. We didn't notice for days. Automation without validation is just automated failure."*

---

## The Spawn-and-Forget Trilogy

### What happened — three times
This failure happened on February 2, February 3, and February 9. Same pattern each time:

1. Alice spawns a sub-agent for a task
2. Alice tells Cordell "I'll check on it"
3. Alice's session gets compacted (context window fills up, old context is summarized)
4. The compacted Alice forgets she was supposed to check
5. The sub-agent finishes its work
6. Nobody tells Cordell
7. Cordell discovers the completed work hours later, or asks and gets a blank stare

**Feb 2:** Builder finished in 22 minutes. Cordell wasn't notified for 3 hours.
**Feb 3:** Dexter completed ShipBob reports at 12:06 AM. Cordell found them at 1:30 AM.
**Feb 9:** Review-polish agent spawned overnight. Session compacted. Nothing delivered. Cordell woke up to silence.

### Why it kept happening
"I'll check on it" is a human promise. It requires continuous attention, memory, and follow-through. LLMs have none of these across session boundaries. Compaction doesn't just reduce context — it destroys intent.

### What changed
Hard rule, no exceptions: **every `sessions_spawn()` call is immediately followed by `cron.add(monitor)`.**

Not "I'll remember to check." Not "I'll set a reminder." A cron job. Automated. Guaranteed to fire. The cron checks the spawned session, and if it's complete, delivers the results.

This is now Rule 7.5 in the operating procedures. It was learned three times because the first two times, we treated it as a one-off mistake instead of a structural problem.

### The lesson
**Intentions don't survive compaction. Automation does.** If the follow-up matters, don't intend to do it — schedule it.

> *Tweetable: "Our AI agent promised to 'check on it' three times. It forgot three times. Now there's a cron job instead of a promise."*

---

## The Meta-Lesson: Why We Publish This

Every system has failures. Most teams hide them behind incident reports that never leave the engineering wiki. We're publishing ours because:

1. **Trust is built on transparency.** If we only showed you the 565 CX conversations and the $3/day operating cost, you'd be right to be skeptical. The failures make the successes credible.

2. **Failures are the product.** Each failure became a rule. Each rule prevents an entire class of future errors. The system's immune system — 18+ incident-derived rules — *is* the product. Without the failures, the rules wouldn't exist, and the system would still be fragile.

3. **This is what production looks like.** If someone tells you their AI agent system "just works," they're either lying or they haven't run it long enough. Production means failures. The question is whether you learn from them or repeat them.

4. **Your failures will be different.** But the *pattern* — failure → analysis → permanent rule → automated enforcement — is universal. That's what we're really selling: not a perfect system, but a system that gets permanently better from every mistake.

---

## Current Failure Rate

As of March 2026:

- **Zero** unmonitored spawns since the hard rule was implemented
- **Zero** wrong-model cron failures since the validation rule
- **Zero** unauthorized vendor closures since the optionality rule
- **One** new failure class discovered per ~2 weeks (and immediately converted to a rule)

The system isn't failure-free. It's failure-*learning*. That's the difference.

> *Tweetable: "A system that never fails isn't resilient. A system that converts every failure into a permanent rule is."*

---

*Every ugly part documented here made the system permanently better. That's not a justification — it's the architecture.*
