---
title: "Reference: Transcript Synthesis Pipeline"
---

# Reference: Transcript Synthesis Pipeline

*How meeting transcripts, podcasts, and articles become structured business intelligence.*

---

## The Problem with Raw Transcripts

A 60-minute coaching call produces 50-70K characters of raw transcript. A podcast produces 30-50K. An article might be 5-10K. The raw text is valuable — but only if it's processed into something actionable.

Every AI meeting tool gives you a summary. "Discussed marketing strategy. Action items: follow up with client." That's worth approximately nothing if it's not connected to the actual work.

Alice's transcript synthesis pipeline doesn't summarize — it **contextualizes**. Every transcript is processed against the full business state: current Notion sprint board, outstanding tasks, recent decisions, active projects, relationship history.

---

## The Pipeline: Raw → Processed → Actionable

### Stage 1: Capture

| Source | Tool | Format | Typical Size |
|--------|------|--------|-------------|
| Business meetings | Granola | Raw transcript + AI summary | 50-70K chars |
| Coaching calls | Granola | Raw transcript + AI summary | 50-70K chars |
| Client calls | Granola | Raw transcript + AI summary | 30-50K chars |
| Podcasts/webinars | Manual import | Transcript file | 30-50K chars |
| Articles | Web fetch | Markdown extraction | 5-15K chars |

**Granola Integration:**
Granola captures meeting transcripts automatically and provides AI-generated summaries. Alice accesses these via the Granola MCP API:

```bash
# Refresh token (tokens expire; refresh script handles rotation)
bash ~/.config/granola/refresh-token.sh

# List recent meetings
list_meetings --time_range "this_week"

# Pull specific transcript
get_meeting_transcript --meeting_id "<uuid>"
```

### Stage 2: Deep Processing

This is where Alice's pipeline diverges from every other meeting tool. The raw transcript is processed against the full business context:

**Context Sources Consulted:**
- Current Notion sprint board (what's already in progress)
- Outstanding tasks across all assignees
- Recent daily memory files (what happened this week)
- Relationship history with meeting participants
- Previous meeting notes with the same people
- Relevant Obsidian vault files (project context, historical decisions)

**Processing Dimensions:**
| Dimension | What Gets Extracted | Why It Matters |
|-----------|-------------------|---------------|
| Action items | Specific tasks with assignees and implied deadlines | Direct workflow input |
| Decisions made | What was decided and the reasoning | Prevents re-litigation in future sessions |
| Homework | Specific deliverables committed to | Accountability tracking |
| Mindset shifts | Strategic reframes, new perspectives | Long-term strategic context |
| Key quotes | Exact words that capture critical insights | Future reference with original voice |
| Outreach targets | People mentioned who should be contacted | Sales/relationship pipeline input |
| Operational changes | Process adjustments discussed | SOP updates |
| Blockers identified | What's preventing progress | Sprint planning input |

### Stage 3: Structured Output

The processed transcript becomes a structured markdown file at `memory/meetings/YYYY-MM-DD-{topic}.md`:

```markdown
# Meeting: [Participants] — [Date]

## Session Context
- Participants, their roles, the meeting's purpose
- Key background (cash flow status, active deals, recent events)

## Key Themes
### Theme 1: [Topic]
- Analysis with direct quotes
- Implications for current work
- Connection to existing tasks/projects

## Action Items & Homework
### Immediate (This Week)
1. [Specific task] — [context from discussion]

### Short-Term (Next 1-2 Weeks)
2. [Specific task]

### Medium-Term (Next Month)
3. [Specific task]

## Strategic Insights
- [Insight with quote and analysis]

## Key Quotes
> "Exact quote from transcript" — [Speaker]
```

### Stage 4: Notion Task Creation

Action items don't just get documented — they become Notion tasks. The Granola-Notion bridge (`scripts/granola-notion-bridge.py`) handles this:

1. **Duplicate detection**: Before creating a task, check the registry — does a similar task already exist? Fuzzy matching on assignee + topic keywords prevents duplicates.
2. **Task creation**: Create the Notion task with title, assignee, priority, deadline, and linked quote from the transcript.
3. **Registry update**: Record the new task in the shared registry so future processing doesn't duplicate it.

```python
# Check for duplicates before creating
result = create_meeting_task_if_new(
    assignee="cordell",
    title="Rewrite positioning deck with new ICP framing",
    topic_keywords=["positioning", "deck", "ICP", "rewrite"],
    meeting_id="8bc5f9cf"
)
```

**Why deduplication matters:** Without it, processing three meetings in a week that all mention "follow up with PrizePicks" would create three separate tasks. The bridge ensures one task exists, enriched with context from all three mentions.

---

## Real Example: Coaching Call Processing

**Input:** Feb 26 coaching call with Hugh Francis (garden3d / Sanctuary Computer). 50K+ character transcript.

**Processing against business context:**
- Cordell in SF, visiting friends (one just joined OpenAI, another raising for fashion-tech AI startup)
- Cash flow extremely tight — pulling credit lines
- VA performance issues (tasks sitting for 2 weeks)
- Active Outlier outbound campaign launching

**Output — Structured meeting notes with:**
- 5 key coaching themes (direct outreach strategy, Big Mario Marketing framework, cultural bridge positioning, content/thought leadership, VA performance standards)
- 10 action items across immediate/short-term/medium-term horizons
- ICP testing framework (startups, movie studios, enterprise brands, AI companies)
- Strategic insights on service offering crystallization ($50-100K elevated merch, $50-100K launch campaigns, $25K go-to-market sprints)
- Key quotes preserved verbatim for future reference

**Notion tasks created:**
- "Escalate VA issue with Magic account manager" → assigned, immediate
- "Write Superculture manifesto/positioning piece" → assigned, next 1-2 weeks
- "Reframe case studies with outcomes" → assigned, next 1-2 weeks
- "Build PDF deck for cold leads" → assigned, next 1-2 weeks, linked to Outlier timeline

---

## Processing Cadence

| Meeting Type | Processing Trigger | Typical Turnaround |
|-------------|-------------------|-------------------|
| Coaching calls | Same day (high priority) | 1-2 hours after call |
| Client calls | Same day | 2-4 hours after call |
| Internal syncs | Next morning | Overnight processing |
| Podcasts/articles | Batch weekly | Nightshift processing |

High-value meetings (coaching, key client calls) get processed immediately. Lower-priority content gets batched into overnight or weekly processing.

---

## The Compound Effect

Meeting notes aren't isolated documents. They feed back into the system:

- **Action items** → Notion tasks → sprint board → daily reviews
- **Decisions** → MEMORY.md updates → future session context
- **Mindset shifts** → SOUL.md or strategic docs → shapes future recommendations
- **Relationship context** → meeting prep for next call → Alice surfaces relevant history

When Alice preps for the next coaching call, she reads all previous coaching notes. She knows what homework was assigned, what was completed, what's still outstanding. The coach doesn't need to repeat context — Alice already has it.

> The pipeline transforms 50K characters of raw conversation into 10 actionable Notion tasks, 5 key quotes, 3 strategic insights, and 1 structured knowledge document — all cross-referenced against the full business state. That's not meeting summarization. That's meeting intelligence.

---

*← Back to [README](../README.md) | Section: [Reference](./) | Related: [Meeting → Action Pipeline](../04-playbooks/meeting-actions.md)*
