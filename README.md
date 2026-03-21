# OpenClaw Memory Stack

> A practical memory stack for AI agents: local typed memory, semantic document retrieval, conversation memory, and health monitoring.

OpenClaw Memory Stack is a reusable pattern for fixing **agent amnesia** without pretending every kind of memory should live in one store.

It separates:

- **local typed memory** for stable facts and preferences
- **document memory** for semantic retrieval over repos, notes, or vaults
- **conversation memory** for raw session history, embeddings, and summaries
- **health checks** so the whole thing does not silently drift out of date

## What this repo is

This repo documents the **generic stack**.

It is intentionally broader than any single deployment. Some implementation details in a real system will vary by host OS, scheduler, embedding provider, database, and alerting channel.

## What TomOS changes

TomOS is one concrete implementation of this stack. In TomOS today, the stack is typically shaped like this:

- **memory-core** stays local on the host
- **document memory** uses PostgreSQL + pgvector
- **conversation memory** stores raw sessions locally and maintains embeddings/summaries for retrieval
- **health checks** watch freshness, drift, and coverage
- **scheduling** often uses LaunchAgents on macOS, with cron as fallback/reference
- **Discord alerting** is usually bot-first, with webhook support still available

That TomOS deployment is a useful reference, but it is **not** the only valid way to use this stack.

## The layers

### 1) Local typed memory

A small always-on store for things that should not require semantic search:

- preferences
- durable facts
- recent history
- rolling metrics

Typical implementation: JSON/JSONL files managed by helper scripts.

### 2) Document memory

Semantic retrieval over documents that change over time but should remain discoverable:

- repo docs
- runbooks
- standards
- vault/notes content

Typical implementation:

- chunk files
- embed the chunks
- store them in PostgreSQL + pgvector
- retrieve by similarity with metadata/policy shaping

### 3) Conversation memory

Retrieval over actual agent/user conversations.

A practical pattern is:

- keep **raw sessions** as the durable source
- generate **embeddings** for retrieval
- maintain **summaries/highlights** for faster continuity and operator review

This is different from document memory. Conversation hits are useful context, but should not automatically outrank canonical docs when exact details matter.

### 4) Health, drift, and coverage

Memory systems fail quietly unless you check them.

Useful health checks include:

- embedding service reachability
- PostgreSQL/pgvector reachability
- document index freshness
- conversation sync freshness
- embedding coverage
- summary coverage
- drift between source sessions and indexed sessions

## Why the separation matters

These layers solve different problems and fail differently:

- typed memory gives fast durable facts
- document memory finds the right file or passage
- conversation memory restores continuity
- health monitoring stops silent rot

Trying to collapse them into a single store usually makes trust worse, not better.

## Quick start

```bash
git clone https://github.com/tomdebres/openclaw-memory-stack.git
cd openclaw-memory-stack
pip install -r requirements.txt
cp .env.example .env
# edit .env for your environment
python scripts/db_init.py
python scripts/memory_health.py
```

See [SETUP.md](SETUP.md) for environment setup details.

## Architecture at a glance

```
┌─────────────────────────────────────────────────────────────┐
│                         AI Agent                            │
└─────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────┐
│                   agent-memory / helpers                    │
└─────────────────────────────────────────────────────────────┘
                              │
          ┌───────────────────┼───────────────────┬───────────────────┐
          ▼                   ▼                   ▼                   ▼
┌────────────────┐  ┌────────────────────┐  ┌──────────────────┐  ┌─────────────────┐
│ Local typed    │  │ Document memory    │  │ Conversation     │  │ Health / drift  │
│ memory-core    │  │ PostgreSQL+pgvector│  │ raw + embeds +   │  │ / coverage      │
│                │  │                    │  │ summaries         │  │                 │
└────────────────┘  └────────────────────┘  └──────────────────┘  └─────────────────┘
```

## Features

| Feature | Description |
|---------|-------------|
| Typed local storage | preferences, facts, history, metrics |
| Semantic document retrieval | chunked docs in PostgreSQL + pgvector |
| Conversation memory | raw sessions plus summaries/embeddings |
| Corpus controls | allowlists, excludes, never-index rules |
| Health monitoring | freshness, drift, coverage, dependency checks |
| Multiple embedding backends | LM Studio, Ollama, OpenAI, or equivalent |
| Scheduler-agnostic ops | LaunchAgents, cron, or another supervisor |

## Documentation

- [ARCHITECTURE.md](ARCHITECTURE.md) — high-level layering and data flow
- [SETUP.md](SETUP.md) — installation and environment setup
- [API.md](API.md) — CLI/reference surface
- [COMPONENTS.md](COMPONENTS.md) — component breakdown
- [EMBEDDINGS.md](EMBEDDINGS.md) — embedding-provider options
- [docs/discord-alerts.md](docs/discord-alerts.md) — Discord alerting patterns
- [SECURITY.md](SECURITY.md) — secret-handling and indexing boundaries

## Requirements

- **Python 3.10+**
- **PostgreSQL + pgvector** for document memory
- **An embedding provider** (LM Studio, Ollama, OpenAI, or similar)
- **Optional**: Discord bot or webhook for alerts

## Used in practice

This stack is used in TomOS, but the repo is written so you can adapt the same pattern to your own OpenClaw deployment or adjacent agent system.

## License

MIT
