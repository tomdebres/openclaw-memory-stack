# CLI Reference

This document covers all command-line tools in the OpenClaw Memory Stack.

---

## Table of Contents

1. [agent-memory](#agent-memory) — Main retrieval CLI
2. [conv-memory](#conv-memory) — Conversation memory
3. [memory_index](#memory_index) — Document indexer
4. [memory_health](#memory_health) — Health checks
5. [memory_discord_alert](#memory_discord_alert) — Discord alerts
6. [index-memory-*](#index-memory-) — Convenience wrappers

---

## agent-memory

Main CLI for querying typed and indexed memory.

### Usage

```bash
agent-memory <command> [options]
```

### Commands

#### query

Search indexed memory using semantic search.

```bash
agent-memory query "search terms"
agent-memory query "planner cron" --limit 5
agent-memory query "memory policy" --source-type repo --path-prefix scripts/
```

| Option | Description |
|--------|-------------|
| `query` | Search terms (required) |
| `--limit` | Number of results (default: 3) |
| `--source-type` | Filter: `vault` or `repo` |
| `--domain` | Filter by domain tag |
| `--path-prefix` | Filter by path prefix |
| `--include-conversations` | Include conversation summaries (default) |
| `--no-conversations` | Exclude conversation memory |

#### recent

Show recent memory events.

```bash
agent-memory recent 20
```

| Option | Description |
|--------|-------------|
| `recent` | Number of events (required) |

#### facts

Show all stored facts.

```bash
agent-memory facts
```

#### preferences

Show all stored preferences.

```bash
agent-memory preferences
```

#### metrics

Show current metrics.

```bash
agent-memory metrics
```

---

## conv-memory

Conversation memory CLI — store, summarize, search, and expand conversations.

### Usage

```bash
conv-memory <command> [options]
```

### Commands

#### store

Store messages from a conversation (reads JSON from stdin).

```bash
conv-memory store <conv_id> < messages.json
```

Example `messages.json`:
```json
[
  {"role": "user", "content": "What's the weather?"},
  {"role": "assistant", "content": "It's sunny today!"}
]
```

#### summarize

Create a DAG summary for a conversation.

```bash
conv-memory summarize <conv_id>
```

#### search

Search conversation summaries.

```bash
conv-memory search "what did we decide about the design"
```

#### expand

Expand a summary back to original messages.

```bash
conv-memory expand <conv_id>
```

#### stats

Show memory statistics.

```bash
conv-memory stats
```

---

## memory_index

Incremental document indexer with policy-driven corpus controls.

### Usage

```bash
python scripts/memory_index.py --source-type <type> --root <path> [options]
```

### Arguments

| Argument | Required | Description |
|----------|----------|-------------|
| `--source-type` | Yes | `vault` or `repo` |
| `--root` | Yes | Root directory to scan |

### Options

| Option | Description |
|--------|-------------|
| `--domain` | Logical domain tag (defaults to source-type) |
| `--policy` | Path to index policy JSON |
| `--full` | Force reindex of all files |
| `--limit` | File limit for smoke testing |
| `--json` | Emit machine-readable JSON output |

### Examples

```bash
# Index a vault
python scripts/memory_index.py --source-type vault --root ~/notes

# Index with custom domain
python scripts/memory_index.py --source-type repo --root ~/projects/myapp --domain myapp

# Smoke test with limit
python scripts/memory_index.py --source-type repo --root ~/code --limit 10

# Full rebuild
python scripts/memory_index.py --source-type repo --root ~/code --full
```

---

## memory_health

Check the health of your memory stack.

### Usage

```bash
python scripts/memory_health.py [options]
```

### Options

| Option | Description |
|--------|-------------|
| `--stale-after-hours` | Hours after which index is considered stale (default: 24) |
| `--json` | Output JSON for bots/cron |
| `--discord` | Output compact Discord-friendly text |

### Examples

```bash
# Human-readable output
python scripts/memory_health.py

# JSON output for monitoring
python scripts/memory_health.py --json

# Discord-friendly output
python scripts/memory_health.py --discord
```

### Output Example (Human)

```
✓ LM Studio reachable (http://localhost:1234)
✓ pgvector reachable
✓ Database initialized (DuckDB)
Documents: 142 | Chunks: 1,203
Last index: 2024-01-15 14:32:00 (2 hours ago)
```

### Output Example (JSON)

```json
{
  "lm_studio_reachable": true,
  "pgvector_reachable": true,
  "db_initialized": true,
  "documents": 142,
  "chunks": 1203,
  "last_index": "2024-01-15T14:32:00",
  "stale": false
}
```

---

## memory_discord_alert

Send health alerts to Discord. Only posts when health is failing or index is stale.

### Usage

```bash
python scripts/memory_discord_alert.py [options]
```

### Options

| Option | Description |
|--------|-------------|
| `--dry-run` | Show what would be posted without sending |
| `--always-post` | Post even when healthy (default: only on problems) |

### Required Environment Variables

| Variable | Description |
|----------|-------------|
| `DISCORD_MEMORY_WEBHOOK_URL` | Your Discord webhook URL |
| `DATABASE_URL` or `OPENCLAW_MEMORY_DSN` | Database connection |
| `LM_STUDIO_URL` | Embedding service URL |

### Examples

```bash
# Preview what would be posted
python scripts/memory_discord_alert.py --dry-run

# Post alerts (only when unhealthy)
python scripts/memory_discord_alert.py

# Always post heartbeat
python scripts/memory_discord_alert.py --always-post
```

---

## index-memory-*

Convenience wrapper scripts for common indexing tasks.

### index-memory-vault

Index a vault (Obsidian-style notes).

```bash
bash scripts/index-memory-vault
bash scripts/index-memory-vault --limit 10  # Smoke test
bash scripts/index-memory-vault --full       # Full rebuild
```

### index-memory-repo

Index a repository.

```bash
bash scripts/index-memory-repo
bash scripts/index-memory-repo --limit 10  # Smoke test
bash scripts/index-memory-repo --full       # Full rebuild
```

Both scripts use environment variables:
- `OPENCLAW_REPO_MEMORY_PATH` — root path to scan
- `OPENCLAW_MEMORY_POLICY_PATH` — path to index policy JSON

---

## Environment Variables

| Variable | Required | Description |
|----------|----------|-------------|
| `DATABASE_URL` | Yes* | Database connection (DuckDB or PostgreSQL) |
| `OPENCLAW_MEMORY_DSN` | Yes* | Alias for DATABASE_URL |
| `LM_STUDIO_URL` | Yes* | LM Studio server URL |
| `LM_STUDIO_EMBED_MODEL` | Yes* | Embedding model name |
| `OLLAMA_URL` | Yes* | Ollama server URL |
| `OLLAMA_EMBED_MODEL` | Yes* | Embedding model name |
| `OPENAI_API_KEY` | Yes* | OpenAI API key |
| `OPENAI_EMBED_MODEL` | Yes* | OpenAI embedding model |
| `DISCORD_MEMORY_WEBHOOK_URL` | No | Discord webhook for alerts |

*At least one embedding service required. DATABASE_URL required for pgvector features.

---

## Exit Codes

| Code | Meaning |
|------|---------|
| 0 | Success |
| 1 | General error |
| 2 | Invalid arguments |
| 3 | Service unreachable |
| 4 | Database error |

---

## See Also

- [SETUP.md](SETUP.md) — Installation guide
- [ARCHITECTURE.md](ARCHITECTURE.md) — High-level design
- [GUIDES/](GUIDES/) — Specific how-tos
