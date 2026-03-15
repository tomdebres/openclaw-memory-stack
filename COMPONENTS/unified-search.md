# Unified Search

CLI and API for querying all memory layers from a single entry point.

## What It Does

Unified Search provides `agent-memory` — a single CLI that queries **all** memory layers:
- Local JSON stores (facts, preferences, history, metrics)
- pgvector-backed documents (when configured)
- Conversation summaries (when configured)

Agents don't need to know about underlying storage. They just query memory, and Unified Search handles the rest.

## How It Works

### The Query Flow

```
1. Parse
   │
   ▼
2. Local lookup ──► facts.json, preferences.json, history.jsonl, metrics.json
   │
   ▼
3. Vector search (if pgvector configured)
   │     • Embed query
   │     • Retrieve similar chunks
   │     • Rank by similarity + metadata
   │
   ▼
4. Merge ── Combine local + vector results
   │
   ▼
5. Output ── Return ranked, formatted results
```

### The Commands

| Command | What It Returns |
|---------|-----------------|
| `query` | Semantic search across documents + conversations |
| `recent` | Recent memory events from history |
| `facts` | All stored environment facts |
| `preferences` | All stored user preferences |
| `metrics` | Current metrics (counters, snapshots) |

### query — Semantic Search

```bash
agent-memory query "search terms"
agent-memory query "planner cron" --limit 5
agent-memory query "memory policy" --source-type repo --path-prefix scripts/
```

**Options:**
| Option | Description |
|--------|-------------|
| `--limit` | Number of results (default: 3) |
| `--source-type` | Filter: vault or repo |
| `--domain` | Filter by domain tag |
| `--path-prefix` | Filter by path prefix |
| `--include-conversations` | Include conversation summaries (default) |
| `--no-conversations` | Exclude conversation memory |

### recent — Recent Events

```bash
agent-memory recent 20
```

Shows the last N events from history.jsonl.

### facts — Environment Facts

```bash
agent-memory facts
```

Returns all facts from facts.json:
- Timezone settings
- Workspace paths
- Stable environment details

### preferences — User Preferences

```bash
agent-memory preferences
```

Returns all preferences from preferences.json:
- Meeting preferences
- Notification settings
- Personal defaults

### metrics — System Metrics

```bash
agent-memory metrics
```

Returns current metrics from metrics.json:
- Rolling counters
- System snapshots

## Why It Matters

### Problem: Fragmented Memory

Without unified search:
- Agents must query different stores manually
- Each store has different APIs
- Easy to miss relevant information
- Results are inconsistent

### Solution: Single Entry Point

Unified Search provides:
- **One command** to query everything
- **Consistent output** format
- **Automatic ranking** — semantic + exact matches
- **Rich filtering** — find exactly what you need

### Result Priority

Results are returned in this order:
1. **Facts** — Exact matches from environment facts
2. **Preferences** — Exact matches from user preferences
3. **History** — Recent events
4. **Vector hits** — Semantic matches from documents/conversations

## Integration: When Agents Should Query

Agents should query memory when:
- User references past conversations
- Answer depends on repo/vault context
- Task spans multiple files or broad topics
- About to ask user something memory might answer

Agents should skip memory when:
- Task is purely local, exact file is open
- User just pasted authoritative source
- Direct inspection is faster

## Example

```bash
# User asks: "What did Tom say about the trading strategy?"

$ agent-memory query "trading strategy"
---
Results (3):
1. [conversation] "We discussed shifting to a momentum-based 
   trading strategy using Python..." (relevance: 0.92)
2. [file: notes/2024-01/trading.md] "The new approach uses 
   technical indicators for entry/exit signals..." (relevance: 0.87)
3. [fact] "trading_strategy: momentum" (relevance: 1.0)
```

The agent now has context from:
- Past conversations
- Relevant documents
- Stored facts

---

See also:
- [agent-memory.py](../scripts/agent-memory.py)
- [retrieval-standards.md](../GUIDES/retrieval-standards.md)
