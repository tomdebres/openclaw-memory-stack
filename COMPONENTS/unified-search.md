# Unified Search

CLI and API for querying all memory layers from a single entry point.

## Overview

Unified Search provides `agent-memory` — a single CLI that queries:
- Local JSON stores (facts, preferences, history, metrics)
- pgvector-backed documents (when configured)
- Conversation summaries (when configured)

## Usage

### agent-memory

```bash
agent-memory <command> [options]
```

### Commands

#### query

Search indexed memory.

```bash
agent-memory query "search terms"
agent-memory query "planner cron" --limit 5
agent-memory query "memory policy" --source-type repo --path-prefix scripts/
```

#### recent

Show recent memory events.

```bash
agent-memory recent 20
```

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
agent-agent memory metrics
```

## Filtering

| Option | Description |
|--------|-------------|
| `--source-type` | Filter by vault or repo |
| `--domain` | Filter by domain tag |
| `--path-prefix` | Filter by path prefix |
| `--include-conversations` | Include conversation summaries (default) |
| `--no-conversations` | Exclude conversation memory |
| `--limit` | Number of results (default: 3) |

## How It Works

1. **Parse** — Parse command and arguments
2. **Local lookup** — Read from JSON stores (facts, preferences, etc.)
3. **Vector search** — If pgvector configured:
   - Embed query
   - Retrieve similar chunks
   - Rank by similarity + metadata
4. **Merge** — Combine local + vector results
5. **Output** — Return formatted results

## Priority

Results are returned in this order:
1. Facts (exact matches)
2. Preferences (exact matches)
3. History (recent events)
4. Vector search hits (semantic matches)

## Integration

Agents should query memory when:
- User references past conversations
- Answer depends on repo/vault context
- Task spans multiple files or broad topics
- About to ask user something memory might answer

Skip memory when:
- Task is purely local, exact file is open
- User just pasted authoritative source
- Direct inspection is faster

---

See also:
- [agent-memory.py](../scripts/agent-memory.py)
- [AGENT_MEMORY.md](../docs/AGENT_MEMORY.md)
