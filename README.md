# OpenClaw Memory Stack

> A typed memory system for AI agents with semantic search, conversation summaries, and automated health monitoring.

OpenClaw Memory Stack fixes **agent amnesia** in OpenClaw. It gives your AI agents long-term memory they can actually trust — combining fast local JSON stores for preferences and facts with pgvector-powered semantic search across documents and conversation history.

## Why Memory Matters

Every good AI assistant starts each conversation fresh — that's by design. But that means:

- **Preferences get forgotten** — "Oh, you prefer morning meetings? I had no idea."
- **Context is lost** — That project decision from last week? Gone.
- **Search is manual** — You end up re-explaining things over and over.

OpenClaw Memory Stack fixes this. It gives your agents structured, queryable memory that persists across sessions — without compromising on privacy or control.

## What It Does

### 🗂️ Typed Local Memory
Four JSON files that survive restarts:
- **preferences.json** — User preferences, settings, defaults
- **facts.json** — Durable environment facts (timezone, workspace paths)
- **history.jsonl** — Append-only log of notable events
- **metrics.json** — Rolling counters and snapshots

### 🔍 Semantic Search
When you connect pgvector + LM Studio (or Ollama/OpenAI), agents can ask natural questions:

```
"What did Tom say about the trading strategy last week?"
```

And get relevant hits from your indexed documents — not just keyword matches, but *meaningful* matches.

### 💬 Conversation Summaries
Automatically store and summarize conversations. Agents can:
- **Search** past summaries for context
- **Expand** a summary back to the original messages
- **Stats** see conversation frequency and trends

### 🏥 Health Monitoring
Automated checks ensure your memory stack stays healthy:
- pgvector connectivity
- LM Studio reachability  
- Index freshness
- Discord alerts when things go wrong

## Quick Start

```bash
# Clone and enter the repo
git clone https://github.com/tomdebres/openclaw-memory-stack.git
cd openclaw-memory-stack

# Install dependencies
pip install -r requirements.txt

# Set up your environment
cp .env.example .env
# Edit .env with your database URL and embedding service

# Initialize the database
python scripts/db_init.py

# Run a health check
python scripts/memory_health.py
```

See [SETUP.md](SETUP.md) for full installation instructions.

## The Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                     Your AI Agent                          │
└─────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────┐
│                  Agent Memory CLI                           │
│         (agent-memory query | recent | facts)              │
└─────────────────────────────────────────────────────────────┘
                              │
            ┌─────────────────┼─────────────────┐
            ▼                 ▼                 ▼
   ┌──────────────┐  ┌──────────────┐  ┌──────────────────┐
   │ Local JSON   │  │   pgvector   │  │ Conversation    │
   │ Memory Store │  │   (optional) │  │    Memory        │
   └──────────────┘  └──────────────┘  └──────────────────┘
                              │
                              ▼
                 ┌──────────────────────┐
                 │  Health Monitor      │
                 │  (cron + Discord)    │
                 └──────────────────────┘
```

## Who Is This For?

- **Builders** running AI agents that need persistent context
- **Developers** who want semantic search across their repos/vaults
- **Power users** tired of re-explaining preferences to every new session

## Features

| Feature | Description |
|---------|-------------|
| Typed local storage | preferences, facts, history, metrics as JSON |
| Semantic search | pgvector-backed document retrieval |
| Conversation memory | Summarize, search, expand past chats |
| Corpus controls | Allowlists, exclude patterns, never-index rules |
| Health monitoring | Automated checks with Discord alerts |
| Multiple embeddings | LM Studio, Ollama, or OpenAI |
| Incremental indexing | Only re-index changed files |

## Documentation

```
openclaw-memory-stack/
├── README.md                    # You're here!
├── SETUP.md                     # Installation guide
├── API.md                       # CLI reference
├── ARCHITECTURE.md              # High-level design diagrams
├── COMPONENTS.md                # Component breakdown
│   ├── file-memory.md          # pgvector indexing
│   ├── conversation-memory.md  # Session storage + summaries
│   ├── health-monitoring.md    # Cron + Discord alerts
│   └── unified-search.md      # CLI + API
├── GUIDES/
│   ├── adding-sources.md      # How to index new sources
│   ├── discord-alerts.md      # Discord integration
│   └── retrieval-standards.md # When agents should query
├── EMBEDDINGS.md               # LM Studio, Ollama, OpenAI
└── SECURITY.md                 # Secrets management
```

## Requirements

- **Python 3.10+**
- **DuckDB** (for pgvector storage)
- **Embedding service**: LM Studio, Ollama, or OpenAI API (at least one)
- **Optional**: Discord webhook for alerts

## Used In

This stack is actively used in [TomOS](https://github.com/tomdebres/tomos) — a personal LifeOS system built on OpenClaw that manages calendars, email, notes, and more.

## License

MIT — use it, fork it, make it yours.

## Related

- [OpenClaw](https://github.com/openclaw/agent) — The agent framework
- [TomOS](https://github.com/tomdebres/tomos) — Personal LifeOS using this memory stack
Updated Sun Mar 15 15:18:36 GMT 2026
