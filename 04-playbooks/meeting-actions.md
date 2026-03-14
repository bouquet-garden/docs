---
title: "Meeting → Action Pipeline Playbook"
---

# Meeting → Action Pipeline Playbook

*Turn 60-minute conversations into structured tasks in 5 minutes.*

---

## Overview

Every meeting generates decisions, action items, and context that needs to be captured and tracked. Most meeting AI tools produce summaries — a paragraph of text that nobody reads a week later.

Alice's meeting pipeline produces structured Notion tasks with linked transcript quotes, deduplicated against existing work, and connected to the broader project context.

---

## The Pipeline

### Step 1: Capture (Automatic)

Meetings are captured by Granola, which records and transcribes automatically. Typical output: 50-70K characters of raw conversation.

### Step 2: Token Refresh

Granola MCP tokens expire periodically. Before pulling transcripts:

```bash
bash ~/.config/granola/refresh-token.sh
```

### Step 3: Pull Transcript

```
# List recent meetings
list_meetings with {"time_range": "this_week"}

# Pull specific meeting transcript
get_meeting_transcript with {"meeting_id": "<uuid>"}

# Pull meeting summary
get_meetings with {"meeting_ids": ["<uuid>"]}
```

Save raw transcript to `/tmp/` for processing.

### Step 4: Process Against Business Context

This is where Alice's pipeline differs from every meeting summary tool. The transcript is processed against:

| Context Source | What It Provides |
|---------------|-----------------|
| **Notion sprint board** | Current tasks, priorities, deadlines |
| **Outstanding tasks** | What's in progress, what's blocked |
| **Recent decisions** | What was already decided (don't re-create) |
| **Active projects** | Project status and stakeholder map |
| **Email threads** | Related correspondence |
| **Previous meeting transcripts** | Continuity from past conversations |

### Step 5: Extract Structured Outputs

The processor identifies five categories:

**1. Homework Items** → Notion tasks
- Specific deliverables mentioned ("rewrite the positioning deck")
- Assigned to the person who committed
- Deadline set (if mentioned)
- Linked to the exact transcript quote where it was discussed

**2. Mindset Shifts** → Reference notes
- Strategic reframing ("stop saying 'design agency' — you're a brand operator")
- Captured as quotes in the meeting notes
- Not actionable tasks — context for future decisions

**3. Outreach Targets** → Tasks with suggested messaging
- People mentioned as contacts to reach
- Task includes: who, why, suggested approach
- Context from the conversation ("Hugh recommended reaching out because...")

**4. Operational Changes** → Calendar events / process updates
- "Move fulfillment review to Monday AM" → calendar event suggested
- "Start sending weekly client updates" → recurring task created

**5. Key Quotes** → Meeting notes
- Significant statements worth preserving
- Linked to speaker and timestamp

### Step 6: Deduplication

Before creating any Notion task, Alice checks existing tasks:
- Fuzzy match on title and description
- If similar task exists → update with new context instead of creating duplicate
- If no match → create as new task with meeting context

**This is critical.** Early versions of the pipeline created 30 tasks from a single coaching call, half redundant. Now it caps at the highest-leverage items and groups related actions.

### Step 7: Create Notion Tasks

Each task includes:
- Clear title
- Description with context from the meeting
- Priority based on discussion emphasis
- Assigned owner
- Due date (if mentioned)
- Link back to transcript with relevant quote highlighted

### Step 8: Send Telegram Summary

```
📋 Meeting Processed: Coaching Call with Hugh Francis

Top Takeaways:
1. Reframe from "design agency" to "brand operator" positioning
2. Three outreach targets identified for March
3. Fulfillment review moving to Monday mornings

Tasks Created (4):
✅ Rewrite positioning deck with new ICP framing (P1, due Mar 14)
✅ Reach out to [Contact 1] re: partnership (P1, suggested messaging attached)
✅ Move fulfillment review to Monday AM (P2, calendar event suggested)
✅ Draft case study from recent project (P2, no deadline)

Mindset Shifts (2):
💡 "You're not competing on design — you're competing on operator efficiency"
💡 "Every client relationship should have a 90-day value target"

[View in Notion] [View Transcript]
```

---

## Coaching Session Variant

For recurring coaching calls (e.g., Hugh Francis, weekly), the Notion task structure is specialized:

```markdown
## Coaching Homework
- [ ] Specific deliverable 1
- [ ] Specific deliverable 2

## Mindset Shifts
- Key reframe 1 (with quote)
- Key reframe 2 (with quote)

## Outreach / Sales
- [ ] Contact 1: reason + suggested messaging
- [ ] Contact 2: reason + suggested messaging

## Content / Positioning
- [ ] Content piece 1
- [ ] Content piece 2

## Operational Changes
- [ ] Process change 1
- [ ] Process change 2

## Key Quotes
> "Quote 1" — Speaker
> "Quote 2" — Speaker

## Next Session Prep
- Topics to revisit
- Progress to report
- Questions to ask
```

The "Next Session Prep" section is generated from incomplete homework items and open questions, ready for the next meeting.

---

## What Makes This Different

### vs. Meeting Summary Tools (Otter.ai, Fireflies, etc.)

| Feature | Summary Tools | Alice Pipeline |
|---------|--------------|----------------|
| Output | Text summary | Structured Notion tasks |
| Context | Meeting only | Meeting + sprint + email + Slack |
| Deduplication | None | Fuzzy match against existing tasks |
| Tracking | Read once, forget | Tasks tracked in sprint |
| Follow-up | None | Next session prep auto-generated |
| Quote linking | Sometimes | Every task linked to source quote |

### vs. Manual Note-Taking

| Aspect | Manual | Alice Pipeline |
|--------|--------|----------------|
| Capture completeness | 30-50% of action items | 90%+ (full transcript) |
| Processing time | 30-60 min | 5 min |
| Deduplication | Rarely done | Automatic |
| Context linking | Mental effort | Automatic cross-reference |
| Follow-through | Often forgotten | Tracked in sprint |

---

## Setting Up

### Prerequisites
- Granola account with MCP API access
- Token refresh script at `~/.config/granola/refresh-token.sh`
- Notion database for meeting notes and tasks
- World state cache active (for cross-reference context)

### First Meeting
1. Have Granola running during meeting
2. After meeting, refresh token
3. Pull transcript and summary
4. Spawn subagent for deep analysis (heavy processing — don't run in main session per Rule 6a)
5. Review created tasks in Notion
6. Review Telegram summary

### Tuning
- **Task count:** Cap at 5-7 highest-leverage items per meeting (early versions over-extracted)
- **Deduplication threshold:** Adjust fuzzy matching sensitivity
- **Priority assignment:** Calibrate based on discussion emphasis, not just keyword matching
- **Quote selection:** Focus on decisions and commitments, not general discussion

---

## Common Issues

**Granola token expiry:** Tokens expire periodically. Always run refresh script before pulling. Symptom: empty or error responses from MCP API.

**Transcript null:** Occasionally Granola returns null transcripts for recordings. Check that the meeting was actually captured. Solution file: `memory/solutions/2026-02-16-meeting-process-granola-transcript-null.md`.

**Over-extraction:** Early pipeline versions created too many tasks. Solution: cap at highest-leverage items, group related actions, use deduplication aggressively.

**[NEEDS CORDELL INPUT]:** Standard meeting types and their expected output structure. Are there meeting types beyond coaching calls that need specialized processing (client check-ins, team standups, vendor calls)?

---

*The meeting pipeline processes coaching calls, business strategy sessions, and sync meetings. Average processing time: 5 minutes for a 60-minute meeting. Output: 4-7 structured Notion tasks with linked quotes + Telegram summary.*

*See also: [Skills & Integrations](../tier1/skills-integrations.md) for Granola details, [Sprint Management](sprint-management.md) for how meeting tasks feed into sprints.*

---

*← Back to [README](../README.md) | Section: [Playbooks](./)*
