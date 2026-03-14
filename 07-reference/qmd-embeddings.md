# Reference: QMD Local Embeddings

*Semantic search that runs entirely on-device. Zero API cost. Zero rate limits. Zero data leaving the machine.*

---

## What It Is

QMD (Quick Memory Database) is the local embedding and search system that powers Alice's semantic memory retrieval. It runs entirely on the Mac — no cloud API calls, no rate limits, no per-query costs, and no data ever leaves the machine.

The system indexes Alice's memory files, Obsidian vault, solution files, and other knowledge sources into a local SQLite database with vector embeddings, enabling natural language search across the entire knowledge base.

---

## Architecture

### Local Models

QMD uses locally-hosted models stored at `~/.cache/qmd/models/`:

| Model | Purpose | Size |
|-------|---------|------|
| `embeddinggemma-300M-Q8_0.gguf` | Text → vector embeddings | 300M params, quantized |
| `Qwen3-1.7B-Q8_0.gguf` | Query understanding, reranking | 1.7B params, quantized |
| `qwen3-reranker-0.6b-q8_0.gguf` | Search result reranking | 600M params, quantized |

All models are GGUF-quantized for efficient CPU/Metal inference on Apple Silicon. No GPU required, though Metal acceleration is used when available.

### Index Storage

The search index lives at `~/.cache/qmd/index.sqlite` — a standard SQLite database with:
- Vector embeddings for every indexed document chunk
- Metadata (source file, chunk position, last modified timestamp)
- Full-text search index as a fallback for exact keyword matches

### How It Works

```
Query: "How did we handle the supplier delay last month?"
  │
  ├── 1. Query embedding (embeddinggemma-300M)
  │     └── Convert natural language → vector
  │
  ├── 2. Vector similarity search (SQLite + cosine similarity)
  │     └── Find top-N most similar document chunks
  │
  ├── 3. Reranking (qwen3-reranker-0.6b)
  │     └── Re-score results for relevance to the specific query
  │
  └── 4. Return ranked results with source file paths and snippets
```

---

## What Gets Indexed

| Source | Path | Content Type |
|--------|------|-------------|
| Daily memory files | `memory/YYYY-MM-DD.md` | Daily logs, decisions, events |
| Long-term memory | `MEMORY.md` | Curated lessons, relationship context |
| Solution files | `memory/solutions/*.md` | Reusable problem-solving patterns |
| Meeting notes | `memory/meetings/*.md` | Processed meeting transcripts |
| Obsidian vault | `/Users/admin/obsidian-vault/` | 1,565+ files of institutional knowledge |
| Documentation | `docs/*.md` | Business docs, pitch packages, analyses |

---

## Why Local Matters

### Zero Cost
Cloud embedding APIs (OpenAI, Cohere, Voyage) charge per token. At 1,565+ vault files plus daily memory files, the indexing cost would be $5-20/month and growing. QMD costs $0 — the models run on hardware Cordell already owns.

### Zero Rate Limits
The Gemini free tier rate limit incident taught a clear lesson: during heavy work sessions, cloud-based search hits rate limits exactly when you need it most. Local embeddings never throttle, never rate-limit, never go down for maintenance.

### Zero Data Exfiltration
Memory files contain private business data — client details, financial information, relationship context, strategic plans. Cloud embedding APIs require sending that text to external servers. QMD never sends a byte off the machine.

### Instant Availability
No network latency. No API key management. No "service temporarily unavailable." The search works offline, on airplane wifi, during ISP outages. If the Mac is running, the search works.

---

## The Gemini Rate Limit Incident

The origin story of local embeddings:

Alice's memory search was originally running on Gemini's free embedding tier. Worked great — until a heavy work session hit rate limits. Searches started failing. Alice couldn't find relevant context from past work. The quality of her responses degraded because she was operating without institutional memory.

Instead of just waiting for the rate limit to reset, Alice built the QMD local fallback. Now the architecture is:

```
Primary: Gemini free tier (when available)
Fallback: QMD local embeddings (always available)
```

In practice, QMD handles nearly all queries locally. The Gemini tier is available as an option but rarely needed. The fallback became the primary.

This is a pattern that repeats across the system: resilience layers that start as fallbacks and become the default because they're faster, cheaper, and more reliable.

---

## Future Considerations

### SuperMemory Integration
SuperMemory (supermemory.ai) offers cloud-based memory management with features like automatic categorization, cross-session memory, and multi-user memory sharing. If OpenClaw scales to multiple users, SuperMemory could provide the shared knowledge layer. For a single-user system, QMD's local approach is superior on every dimension (cost, speed, privacy).

### Convex Backend
Convex (convex.dev) offers real-time reactive database infrastructure. For a future where Alice's memory needs to sync across multiple devices (Mac + iPhone companion app + potential cloud deployment), Convex could provide the real-time sync layer while QMD handles local search.

### Scaling Considerations
Current index size is manageable on a single machine. At 10,000+ files, indexing time and search latency would need benchmarking. Options:
- Incremental indexing (only re-index changed files)
- Chunk-level caching (don't re-embed unchanged chunks)
- Tiered search (search recent files first, expand to vault only if needed)

> Local embeddings aren't a compromise — they're an advantage. Zero cost, zero rate limits, zero data leaving the machine, instant availability. The model that runs on your hardware is the model that never lets you down.

---

*← Back to [README](../README.md) | Section: [Reference](./) | Related: [Memory Architecture](../02-deep-technical/memory-architecture.md), [Model Resilience](../02-deep-technical/model-resilience.md)*
