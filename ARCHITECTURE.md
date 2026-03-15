# Architecture

High-level design and data flow for the TomOS Memory Stack.

## Overview

```
┌──────────────────────────────────────────────────────────────────┐
│                         Your AI Agent                            │
│                      (Concierge, Builder, etc.)                  │
└──────────────────────────────────────────────────────────────────┘
                                │
                                ▼
┌──────────────────────────────────────────────────────────────────┐
│                    Agent Memory CLI                              │
│         `agent-memory query | recent | facts`                  │
└──────────────────────────────────────────────────────────────────┘
                                │
          ┌─────────────────────┼─────────────────────┐
          ▼                     ▼                     ▼
┌─────────────────┐  ┌─────────────────┐  ┌──────────────────────┐
│  Local JSON    │  │    pgvector    │  │   Conversation      │
│  Memory Store  │  │   (optional)   │  │     Memory           │
│                 │  │                 │  │                      │
│ - preferences  │  │ - documents    │  │ - summaries         │
│ - facts        │  │ - chunks       │  │ - messages          │
│ - history      │  │ - index_runs   │  │ - stats             │
│ - metrics      │  │                 │  │                      │
└─────────────────┘  └─────────────────┘  └──────────────────────┘
                                │
                                ▼
┌──────────────────────────────────────────────────────────────────┐
│                    Health Monitor                                │
│         `memory_health.py` + `memory_discord_alert.py`          │
└──────────────────────────────────────────────────────────────────┘
```

## Data Flow

### 1. Retrieval (Agent → Memory)

When an agent queries memory:

1. CLI receives the query
2. Local JSON stores are read (facts, preferences, history, metrics)
3. If pgvector is configured:
   - Query is embedded via LM Studio/Ollama/OpenAI
   - Similar chunks are retrieved from pgvector
   - Results are ranked by similarity + metadata boosts
4. Results are returned to the agent

### 2. Indexing (Files → pgvector)

When documents are indexed:

1. Indexer scans the root directory
2. Files are filtered against the policy (allow/exclude/never-index)
3. Each file is chunked (by heading or paragraph)
4. Chunks are embedded via the configured embedding service
5. Chunks are stored in pgvector with metadata

### 3. Conversation Memory

Conversations are stored separately:

1. Messages are stored raw in `conversation_messages`
2. A DAG summary is generated (condensed view)
3. Summaries are embedded and indexed
4. Agents can search summaries or expand to full context

## Storage Layers

### Layer 1: Local JSON (Always On)

| File | Purpose | Format |
|------|---------|--------|
| `preferences.json` | User preferences | JSON |
| `facts.json` | Environment facts | JSON |
| `history.jsonl` | Event log | JSON Lines |
| `metrics.json` | Counters/snapshots | JSON |

Location: `.staging/memory/` (configurable)

### Layer 2: pgvector (Optional)

When enabled, provides semantic search:

| Table | Purpose |
|-------|---------|
| `memory_documents` | Document metadata |
| `memory_chunks` | Embedded text chunks |
| `memory_index_runs` | Index run history |

### Layer 3: Conversation Memory

| Table | Purpose |
|-------|---------|
| `conversation_messages` | Raw messages |
| `conversation_summaries` | DAG summaries |

## Embedding Pipeline

```
File → Read → Chunk → Embed → Store in pgvector
           │
           ▼
    ┌──────────────┐
    │ Chunk Policy │
    │              │
    │ - allow      │
    │ - exclude    │
    │ - never_index│
    └──────────────┘
```

## Health Monitoring

1. **On-demand**: `memory_health.py` checks:
   - LM Studio/Ollama reachability
   - pgvector connectivity
   - Document/chunk counts
   - Index freshness

2. **Scheduled**: `memory_discord_alert.py` runs via cron:
   - Checks health
   - Posts to Discord only when:
     - Health is failing
     - Index is stale
     - Or `--always-post` is set

## Security Boundaries

The `never_index` policy prevents sensitive content from being embedded:

- `.env*` files
- Private keys, certificates
- Password databases
- Files under `secrets/` or `.secrets/`
- Credential-bearing filenames (`*token*`, `*password*`, etc.)

See [SECURITY.md](SECURITY.md) for details.

## Extensibility

The modular design allows:

- **New embedding providers** — just add a new adapter
- **New storage backends** — swap DuckDB for PostgreSQL
- **New index sources** — add a new `--source-type`
- **New alert channels** — extend `memory_discord_alert.py`

---

See [COMPONENTS.md](COMPONENTS.md) for detailed component documentation.
