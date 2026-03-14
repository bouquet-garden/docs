---
title: "Reference: Knowledge Vault — Obsidian Integration"
---

# Reference: Knowledge Vault — Obsidian Integration

*1,565+ files in a PARA-structured vault. The long-term knowledge layer that gives Alice institutional memory beyond daily notes.*

---

## What It Is

Alice's knowledge system extends beyond her daily memory files into a full Obsidian vault at `/Users/admin/obsidian-vault`. This vault contains 1,565+ markdown files organized using the PARA methodology — Projects, Areas, Resources, Archives — and serves as the deep knowledge layer for the entire system.

The vault isn't just storage. It's queryable, indexed, and integrated into Alice's workflow through the `obsidian-cli` tool and QMD local embeddings.

---

## Structure: PARA Methodology

| Folder | Purpose | Contents |
|--------|---------|----------|
| `01 Projects/` | Active, time-bound work with defined outcomes | Client projects, product launches, campaigns, active deals |
| `02 Areas/` | Ongoing responsibilities with standards to maintain | Business operations, finance, brand management, partnerships |
| `03 Resources/` | Topics of interest, reference material | Industry research, competitive intelligence, frameworks, templates |
| `04 Archives/` | Completed projects and inactive material | Finished client work, past campaigns, deprecated processes |

### Why PARA Works for Agent Systems

PARA's progressive filing (active → inactive) maps naturally to how an agent prioritizes information:

- **Projects** are checked first for current work context
- **Areas** provide ongoing operational knowledge
- **Resources** supply reference material for research and analysis
- **Archives** serve as historical record when similar situations arise

When Alice needs to research a topic, she doesn't search 1,565 files flat. The PARA structure provides natural prioritization: current projects first, then ongoing areas, then resources, then archives.

---

## How Alice Uses the Vault

### Direct File Access
Alice reads vault files directly when she needs deep context. Examples:

- Before a client meeting: reads the client's project folder for history, past deliverables, relationship notes
- During contract review: reads resource files on legal frameworks, past contract analyses
- For competitive research: reads industry analysis files, competitor profiles, market reports
- For pitch preparation: reads past pitch packages, case studies, and positioning documents

### Obsidian CLI Integration
The `obsidian-cli` tool provides programmatic access:

```bash
# Search vault contents
obsidian-cli search "woven label sourcing"

# Read specific notes
obsidian-cli read "01 Projects/Superculture/Q1-2026-Production.md"

# List notes in a folder
obsidian-cli list "02 Areas/Operations"
```

### QMD Semantic Search
For fuzzy queries where exact keywords might not match, QMD local embeddings (see [QMD Embeddings](./qmd-embeddings.md)) index the vault for semantic search. "How did we handle the last supplier delay?" returns relevant notes even if none contain those exact words.

---

## What's In the Vault

### Projects (Active)
- Client project files with scope, timeline, deliverables, and communication history
- Product launch plans with production schedules and vendor contacts
- Campaign strategies with creative briefs and performance targets

### Areas (Ongoing)
- **Operations:** SOPs, vendor relationships, production workflows
- **Finance:** Budget tracking, invoice templates, payment schedules
- **Brand:** Positioning documents, style guides, tone documentation
- **Partnerships:** Partner profiles, deal structures, relationship history

### Resources (Reference)
- **Industry Research:** Market reports, trend analyses, competitive intelligence
- **Frameworks:** Business models, pricing strategies, negotiation playbooks
- **Templates:** Email templates, contract templates, pitch templates
- **Learning:** Notes from books, podcasts, articles, and courses

### Archives (Historical)
- Completed project documentation
- Past campaign results and retrospectives
- Deprecated processes (kept for reference)

---

## The Knowledge Compound Effect

The vault isn't static. It grows with every project, every meeting, every research session:

1. **Meeting transcripts** get processed into structured notes with action items, decisions, and key quotes → filed into the relevant project or area folder
2. **Project completions** generate retrospective notes → filed into archives with lessons learned
3. **Research sessions** produce structured analysis → filed into resources for future reference
4. **Solution files** from Alice's problem-solving → cross-referenced with relevant vault notes

Over time, the vault becomes an institutional knowledge base that no human team could maintain at this depth. When Alice starts a new sourcing project, she doesn't just access her memory files — she accesses every sourcing-related note in the vault: past supplier evaluations, industry terminology files, negotiation outcomes, and production timelines.

---

## Integration Architecture

```
┌─────────────────────────────────────────────┐
│              Alice's Knowledge Stack         │
├─────────────────────────────────────────────┤
│ Layer 4: Compound Learning (weekly synthesis)│
│ Layer 3: Solutions Library (memory/solutions)│
│ Layer 2: Long-Term Memory (MEMORY.md)        │
│ Layer 1: Daily Notes (memory/YYYY-MM-DD.md)  │
├─────────────────────────────────────────────┤
│ Deep Layer: Obsidian Vault (1,565+ files)    │
│   └── Indexed by QMD local embeddings        │
│   └── Accessible via obsidian-cli            │
│   └── PARA-organized for priority access     │
└─────────────────────────────────────────────┘
```

The vault sits beneath the 4-layer memory architecture as the deep knowledge reservoir. Daily notes capture what happened today. MEMORY.md captures curated long-term context. Solutions capture reusable patterns. The vault captures *everything else* — the full institutional knowledge of the business.

> 1,565+ files of institutional knowledge, PARA-organized, semantically indexed, and accessible in milliseconds. This is the knowledge moat that compounds every week — and it's the layer that makes Alice's analysis qualitatively different from any chatbot that starts every conversation from zero.

---

*← Back to [README](../README.md) | Section: [Reference](./) | Related: [Memory Architecture](../02-deep-technical/memory-architecture.md)*
