# The Delegation Philosophy

**Managing AI is managing. The skills transfer perfectly.**

Alice isn't a tool you use. She's a team member you manage. And the management philosophy that makes her effective isn't novel — it's the same framework that works for human teams, applied with AI-specific constraints.

This document is the management playbook. Not the technical architecture (that's in `01-foundations/`). The *philosophy* — when to trust, when to verify, when to delegate, when to override.

---

## The Core Principle: External = Ask, Internal = Act

This is the single most important delegation rule in Alice's operating system, and it took three failures to get right.

### External actions require approval
Anything that leaves the system — emails to clients, messages to vendors, tweets, Slack messages to external channels — Alice drafts but doesn't send. She surfaces the draft with context and waits for approval.

Why: because external actions have consequences you can't undo with `git revert`. A wrong email damages a relationship. A premature vendor decline closes doors (we learned this one the hard way — see `failure-catalog.md`).

### Internal actions are autonomous
File organization, memory updates, cron management, sprint status changes, research, analysis, code generation on branches — Alice acts freely. These are reversible. If she organizes files wrong, we reorganize. If she writes bad code, it's on a branch.

### The gray zone
Some actions blur the line. Updating a Notion task that a client can see? Creating a calendar event that sends a notification? Marking an email as read?

The rule: **if in doubt, it's external.** Better to over-ask than under-ask. Trust is earned incrementally. As Alice demonstrates good judgment on edge cases, specific gray-zone actions get promoted to autonomous.

> *Tweetable: "The delegation rule that makes AI agents safe: external actions = ask permission, internal actions = act freely. One line. Zero ambiguity."*

---

## Expensive vs. Cheap: The Model Routing Philosophy

### The human parallel
You don't pay your senior director to check if the mail arrived. You don't ask your intern to negotiate a $500K contract. Different tasks require different levels of judgment, and you pay accordingly.

### The AI implementation

| Task Type | Model | Cost | Why |
|-----------|-------|------|-----|
| **Coordination decisions** | Claude Opus | $0.075/1K tokens | Alice decides what to escalate, what to delegate, what to act on. Bad judgment here cascades everywhere. |
| **Implementation** | Sonnet / Kimi K2.5 | $0.015/1K tokens | Dexter writes code. Mojo designs. Good enough for execution with review gates. |
| **Monitoring** | Claude Haiku | $0.001/1K tokens | "Any new emails?" "Any Slack mentions?" Simple checks, run frequently, must be cheap. |
| **Search/embeddings** | Gemini free tier | $0 | Semantic search doesn't need frontier reasoning. Free is fine. |

### The math that makes it work
47 crons on Opus: ~$15/day. 47 crons on Haiku: ~$0.50/day. Alice on Opus: ~$2/day. Total: ~$3/day.

The 30x cost difference between "every task gets the best model" and "each task gets the right model" is the difference between "interesting experiment" and "production-viable."

### The lesson from free-tier models
We tried running crons on GLM-4.7-free. Zero cost. ~50% failed silently with rate limit errors. We didn't notice for days. Free models are for experiments. The cheapest reliable model (Haiku at $0.001/run) is infinitely cheaper than a free model that silently fails.

> *Tweetable: "Opus for thinking. Haiku for checking. Free tier for searching. $3/day. The AI budget isn't about what the model can do — it's about what the task needs."*

---

## Spawn vs. Self-Execute: When to Create Sub-Agents

### The problem
Alice has a conversation going with Cordell. A task comes up that requires multiple API calls, file generation, cache warming, or anything that takes more than 2 tool calls.

Should Alice do it herself, or spawn a sub-agent?

### Rule 6a: Never run build tasks in the main session

**The main session is the operator's interface.** It's where decisions happen, priorities get discussed, quick lookups are performed. It is not a build terminal.

If the task involves: multiple API calls, file generation loops, bulk fetching, script testing with retries, or anything that takes more than 2 tool calls — **spawn a sub-agent.**

**The incident that created this rule:** Alice built an entire Granola cache layer directly in the main Telegram session. Every curl command, every cache miss, every stat check appeared as Telegram noise. The operator saw raw HTTP responses instead of conversation. Should have been one line: "Build Granola cache layer per this spec."

**The test:** Before running a tool, ask: "Would the operator want to see this in Telegram?" If no → sub-agent.

### Rule 6: Never race a sub-agent on the same file

If Alice has already done the research — read the files, analyzed the patterns, has the context loaded — she should write the deliverable herself. Don't spawn a sub-agent to write something from a summary of what you already know.

**The incident:** Alice deep-dived a codebase (read 15+ files, analyzed patterns), then spawned a Sonnet sub-agent to "write the spec." The sub-agent worked from a prompt summary, not from Alice's loaded context. It produced 2,006 lines with fabricated code examples. Meanwhile, Alice wrote 1,306 lines with real code from the actual dive. Both targeted the same file.

**The rules:**
1. If you have the context, YOU write it
2. Never have two writers targeting the same file path
3. Sub-agents work from prompts, not from your memory — quality degrades on translation

### Rule 6b: Never defer urgent analysis during active sessions

When the operator sends data during an active work session — a CSV, a screenshot, a file — start processing immediately. Spawn a sub-agent if it's heavy. Never say "tomorrow."

**The incident:** Cordell sent an 875-contact CSV for analysis. Alice saved the file and said "Tomorrow: I analyze the contacts." Cordell was in execution mode. "Tomorrow" was unacceptable. 30 minutes wasted.

> *Tweetable: "Three rules for AI delegation: don't build in the chat window, don't spawn what you already know, and never say 'tomorrow' to a founder in execution mode."*

---

## The Management Model

### How you'd manage a human team
1. **New hire:** Watch everything, approve everything, correct frequently
2. **3 months in:** Trust routine work, review exceptions, course-correct weekly
3. **Veteran:** Delegate outcomes, review results, intervene on strategy

### How Alice's trust level works
The exact same progression, accelerated:

**Week 1-2 (New hire):**
- Review every email draft before sending
- Approve every vendor communication
- Check every cron output manually
- Correct judgment calls explicitly

**Week 3-5 (Building trust):**
- Approve email drafts only for new contacts or sensitive topics
- Let routine vendor follow-ups go autonomously
- Review cron outputs weekly instead of daily
- Start delegating research and analysis without review

**Week 6+ (Earned autonomy):**
- Most internal operations are fully autonomous
- External communications reviewed only for high-stakes contacts
- Overnight work happens without pre-approval (budget-capped)
- Alice escalates to Cordell only when she genuinely needs judgment

### The trust calibration mechanism
Every failure tightens trust. Every success loosens it. The incident-derived rules are the explicit record of trust calibrations.

Rule 12 ("Never claim no access without checking") exists because Alice was lazy once. That laziness cost trust. Now she checks before claiming.

Rule 7.5 ("Every spawn gets a monitoring cron") exists because Alice forgot to follow up three times. That forgetfulness cost trust. Now it's automated.

The rules aren't punishment. They're the same thing that happens on any team: when someone drops the ball, you add a process to prevent it from happening again. The process stays until they've demonstrated it's no longer needed.

---

## The Delegation Hierarchy

### Level 1: Full autonomy (no approval needed)
- File organization and memory management
- Research and analysis
- Internal code on feature branches
- Cron management and optimization
- Sprint status updates
- Morning briefing compilation

### Level 2: Draft and present (approval before action)
- Client-facing emails
- Vendor communications
- Social media posts
- Calendar changes that notify others
- Expense or purchasing decisions

### Level 3: Recommend only (human decides and executes)
- Hiring / team changes
- Pricing decisions
- Strategic direction
- Legal responses
- Anything involving money over threshold

### Level 4: Don't even draft
- Responses to legal threats
- Communications during active disputes
- Anything the operator has explicitly flagged as personal

---

## The 3-Strike Kill Switch

When a sub-agent fails the same task 3 times:

1. **Stop spawning.** Don't try a 4th time hoping it works.
2. **Write up what's failing.** Not "it didn't work" — specifically what went wrong and why.
3. **Escalate to the operator.** With the write-up. Let the human decide whether to change approach, change model, or abandon the task.
4. **Never burn tokens hoping attempt #4 works.** That's not persistence. That's waste.

This is the AI equivalent of the management principle: if you've told someone how to do something three times and they can't do it, the problem isn't effort — it's capability or approach. Change one of those, or reassign.

---

## The Philosophy, Summarized

| Human Management Concept | AI Implementation |
|--------------------------|-------------------|
| Senior staff get harder assignments | Opus for judgment, Haiku for monitoring |
| New hires need more oversight | Early weeks: review everything |
| Trust is earned, not given | Autonomy expands as judgment proves reliable |
| External communication needs approval | External = ask, internal = act |
| Don't micromanage routine work | Crons handle monitoring autonomously |
| Escalate what's above your pay grade | 3-tier escalation: agent → Alice → Cordell |
| Document processes after mistakes | Incident → rule → enforcement |
| Don't let one person become a bottleneck | 4 agents with clear domains |

The gap between "managing humans" and "managing AI agents" is smaller than you think. The skills transfer. The intuitions transfer. The hard-won lessons about trust, delegation, and escalation — they all transfer.

The operators who succeed with AI agents won't be the ones who understand the technology best. They'll be the ones who understand management best.

> *Tweetable: "Managing AI agents isn't a technical skill. It's a management skill. If you can run a team, you can run an AI team. The intuitions are the same."*

---

*This philosophy isn't static. It evolves as Alice earns more trust, as the system handles more edge cases, and as the operator's comfort with delegation grows. The framework stays the same — the calibration changes.*
