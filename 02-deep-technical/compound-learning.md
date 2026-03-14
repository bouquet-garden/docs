# Compound Learning Engine

*How every failure makes the system permanently better.*

---

## The Core Idea

Most AI systems make the same mistakes forever. Every session starts fresh, every error is forgotten, every lesson evaporates when the conversation ends.

Alice's compound learning engine breaks this cycle. Every significant failure gets documented, analyzed, and encoded into enforceable rules that are read at the start of every session. The system that exists today is literally the accumulated residue of every mistake that's been made and learned from.

> This isn't a feature — it's an architecture. And it's the single most important differentiator between Alice and any other AI agent deployment.

---

## The Loop

```
Incident occurs
    ↓
Immediate documentation (daily notes)
    ↓
Root cause analysis (what happened, why, what could prevent it)
    ↓
Rule creation (specific, enforceable, with context)
    ↓
Promotion to AGENTS.md (read every session)
    ↓
Behavioral change (system acts differently next time)
    ↓
Verification (did the rule prevent recurrence?)
    ↓
Weekly synthesis (cross-session pattern recognition)
    ↓
System updates (AGENTS.md, SOUL.md, TOOLS.md)
    ↓
Next week's agents start smarter than last week's
```

**The key insight:** This isn't manual training. It's not prompt engineering. It's emergent institutional learning — the same way organizations develop operating procedures from incident reports. Every rule in AGENTS.md has a story behind it. Every story is a failure that will never happen again.

---

## How Rules Get Born

### Step 1: Something Goes Wrong

Not a theoretical failure — a real one. With real consequences.

**Example — Rule 12:** On February 28, Cordell asked Alice about Notion integration. Alice responded: "I don't have direct Notion API integration." This was wrong. The API key existed at `~/.config/notion/api_key`. Four scripts used it daily. Alice guessed instead of checking.

### Step 2: Document the Incident

In daily notes, with timestamps and exact details:

```
2026-02-28 14:32 — Told Cordell "I don't have Notion API integration."
False. Key exists at ~/.config/notion/api_key. 4+ scripts use it.
Root cause: laziness — guessed instead of checking filesystem.
```

### Step 3: Create the Rule

Specific, enforceable, with the incident story embedded:

```markdown
## 🚫 Rule 12: Never Claim "No Access" Without Checking
NEVER say "I don't have access to X" without checking first.

Rules:
1. Before saying "I can't access X" → run `which X`, `ls ~/.config/X`
2. Check the Integration Registry in TOOLS.md
3. If genuinely not available → say "I checked and couldn't find credentials"
4. Guessing "no" is worse than spending 10 seconds verifying
```

### Step 4: Promote to AGENTS.md

The rule goes into AGENTS.md, which is read at the start of every session. It's not optional reading — it's the first thing Alice does. The rule becomes permanent institutional knowledge.

### Step 5: Verify

The next time a similar situation arises, the rule fires. Alice checks before claiming no access. The failure category is eliminated.

---

## The Weekly Synthesis

Every week, a compound learning synthesis reviews the entire week's activity across all agents:

**What gets reviewed:**
- All agent learnings files
- Daily notes from the week
- Solution files created
- Incidents and their resolutions
- Cron performance and failures
- Infrastructure health reports

**What gets produced:**
- Cross-cutting patterns (e.g., "three agents hit the same IMAP failure")
- Recommended system updates (with priority: MUST FIX vs SHOULD FIX)
- Model performance observations
- Repeated failures to break
- Operational metrics
- Open proposals from previous weeks (tracking implementation)

**Real output from W09-W11 synthesis:**

| Week | Key Finding | Action Taken |
|------|-------------|-------------|
| W09 | Email pre-send checklist missing (Annie could accidentally send instead of draft) | Proposed for SOUL.md |
| W09 | 22 broken crons discovered, all migrated to Haiku | Completed in-week |
| W10 | W09 proposals not yet implemented | Flagged as carryover risk |
| W10 | Gateway memory leak detected (2.1GB, possible leak) | Flagged for Phoenix |
| W10 | Nightshift V2 executing cleanly (6 tasks, 77% budget utilization) | Validated |
| W11 | Credential cascade failure: IMAP auth breaks entire escalation chain | Added proactive monitoring |
| W11 | Cordell at 100% IC ratio — the primary business blocker | Flagged for team structure discussion |

**The meta-pattern:** W10 explicitly tracked that W09's proposals weren't implemented. The synthesis itself has a feedback loop — it doesn't just generate recommendations, it tracks whether previous recommendations were acted on.

---

## The Solutions Library

Beyond rules, the compound learning system produces reusable solutions — documented patterns for recurring problem types.

**Current inventory (20+ files):**

| Solution | Category | What It Captures |
|----------|----------|-----------------|
| Message delivery guarantees | Operations | 3-layer defense against eaten reply slots |
| Telegram error noise prevention | Operations | How to stop raw exec errors from reaching users |
| iOS app build pipeline | Technical | Spec → Build → Verify → Ship pattern |
| Meeting process patterns | Operations | 6+ variations for different meeting types |
| Overnight batch execution | Operations | Budget management, task scoring, domain rotation |
| Heartbeat pipeline dedup | Technical | Preventing duplicate alerts across cron runs |

Each solution file follows a standard template:
```yaml
category: technical|operations|communications
difficulty: simple|medium|complex
tags: [searchable, keywords]
outcome: success|partial|failure
```

**Before starting any significant task,** Alice searches the solutions library:
```bash
find memory/solutions/ -name "*.md" -exec grep -l "relevant-keyword" {} \;
```

This "check before building" discipline means solutions compound — each problem solved makes similar future problems faster.

---

## The Four Learning Domains

Every agent captures learnings across four domains:

| Domain | Examples |
|--------|---------|
| **Operational** | "Config patches replace arrays — must include all items" |
| **Knowledge** | "Damask weave is smoother than satin weave for woven labels" |
| **Business** | "PrizePicks exiting Canada — pivot to US-only strategy" |
| **Personal** | "Cordell prefers bullet points on Telegram, not paragraphs" |

These aren't just stored — they're surfaced. Frank's Wednesday Ops Health Review checks: Are agents writing learnings? Are they tagged by domain? Are any flagged for SOUL.md integration that haven't been applied?

---

## Compound Learning vs. Fine-Tuning

| Approach | Compound Learning | Fine-Tuning |
|----------|-------------------|-------------|
| Speed | Instant (add a rule, it's live next session) | Hours to days (collect data, train, deploy) |
| Explainability | Every rule has a story you can read | Black box weight adjustments |
| Reversibility | Delete the rule | Re-train without the data point |
| Specificity | "Never claim no access without checking" | General behavioral shift |
| Cost | $0 (just editing a markdown file) | GPU hours for training runs |
| Compounding | Rules reference other rules, build on each other | Diminishing returns on incremental training |

> Compound learning is the poor man's fine-tuning — except it's faster, cheaper, explainable, reversible, and more specific. The only thing fine-tuning has going for it is that it doesn't require reading a file at session start.

---

## Measuring Improvement

The clearest measure: compare Alice's error rate over time.

- **Week 1-2:** Spawn-and-forget (3 incidents of the same pattern). Claims of no access without checking. Sending to wrong contacts. Closing vendor optionality.
- **Week 3-4:** Each of those became a rule. New errors emerged but old categories stopped.
- **Week 5-8:** Compound synthesis catching cross-agent patterns. Infrastructure stabilizing.
- **Week 9-11:** System self-auditing. Synthesis tracking whether previous recommendations were implemented.

The error categories don't recur. New ones emerge, but the population of "ways the system can fail" is monotonically shrinking. That's compound learning working.

---

*← Back to [README](../README.md) | Related: [Incident-Derived Rules](incident-rules.md) · [Memory Architecture](memory-architecture.md)*
