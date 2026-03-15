# Components

The TomOS Memory Stack is built from four main components, each solving a specific memory challenge:

| Component | What It Does | Why It Matters |
|-----------|--------------|----------------|
| [File Memory](COMPONENTS/file-memory.md) | pgvector-backed document indexing with embeddings | Semantic search across your vaults and repos |
| [Conversation Memory](COMPONENTS/conversation-memory.md) | Full conversation storage + DAG summaries | Never lose context between sessions |
| [Health Monitoring](COMPONENTS/health-monitoring.md) | Cron jobs + Discord alerts | Proactive awareness when things break |
| [Unified Search](COMPONENTS/unified-search.md) | Single CLI for all memory layers | Agents can query everything from one place |

---

## File Memory

**What it does:** Indexes documents from vaults (Obsidian-style notes) and repositories using pgvector, making them searchable via semantic similarity.

**How it works:**
1. **Scan** — Walk the root directory, collect all files
2. **Filter** — Apply policy rules (allow/exclude/never-index)
3. **Chunk** — Split files into semantic chunks (headings, paragraphs)
4. **Embed** — Convert chunks to vectors via LM Studio/Ollama/OpenAI
5. **Store** — Save to pgvector with rich metadata

**Why it matters:**
- Agents can find relevant documents without exact keywords
- "What did I write about project X?" returns meaningful results
- Incremental indexing means only changed files are re-embedded

**Key features:**
- Policy-driven corpus controls (allow/exclude/never-index)
- Rich metadata: project, domain, path, tags
- Incremental updates (only re-index changed files)

---

## Conversation Memory

**What it does:** Stores full conversation messages, generates condensed DAG summaries, and enables semantic search across past chats.

**How it works:**
1. **Store** — Raw messages saved to SQLite with timestamps
2. **Summarize** — DAG-structured summaries capture key decisions, facts, actions
3. **Embed** — Summaries get vector embeddings for search
4. **Search** — Natural language queries find relevant past conversations
5. **Expand** — Summaries can expand back to original messages when needed

**Why it matters:**
- No more "I don't remember discussing that"
- Agents have full context from previous sessions
- Search finds relevant past discussions, not just exact keyword matches
- Expandability means summaries are safe — detail is preserved

**Key features:**
- Full message storage (not just summaries)
- DAG-structured summaries for hierarchy
- Automatic inclusion in unified search

---

## Health Monitoring

**What it does:** Automated health checks ensure your memory stack stays operational, with Discord alerts when things go wrong.

**How it works:**
1. **Check** — Verify embedding service reachability
2. **Connect** — Test pgvector connectivity
3. **Count** — Verify documents and chunks exist
4. **Freshness** — Confirm index was recently updated
5. **Alert** — Post to Discord if any checks fail

**Why it matters:**
- You know immediately when embedding service goes down
- Stale indexes are detected before they cause problems
- Proactive monitoring prevents silent failures

**Key features:**
- LM Studio/Ollama reachability checks
- Configurable stale thresholds
- Discord webhook integration
- Multiple output formats (human, JSON, Discord)

---

## Unified Search

**What it does:** Provides `agent-memory` — a single CLI that queries all memory layers from one entry point.

**How it works:**
1. **Parse** — Interpret command and arguments
2. **Local lookup** — Read from JSON stores (facts, preferences)
3. **Vector search** — Query pgvector for semantic matches
4. **Merge** — Combine results from all sources
5. **Output** — Return ranked, formatted results

**Why it matters:**
- Agents don't need to know about underlying storage
- Single query interface for everything
- Results ranked by relevance, not just type

**Key features:**
- Single entry point: `agent-memory query`
- Commands: query, recent, facts, preferences, metrics
- Rich filtering: source-type, domain, path-prefix
- Conversation inclusion/exclusion options

---

## How They Work Together

```
User Question
       │
       ▼
┌─────────────────────────────────┐
│     agent-memory query         │
│  "What did Tom say about X?"   │
└─────────────────────────────────┘
       │
       ├──────────────────┬──────────────────┐
       ▼                  ▼                  ▼
┌─────────────┐   ┌─────────────┐   ┌─────────────┐
│ Local JSON  │   │ File Memory │   │ Conversation│
│ (facts,     │   │ (pgvector)  │   │ Memory      │
│ preferences)│   │             │   │             │
└─────────────┘   └─────────────┘   └─────────────┘
       │                  │                  │
       └──────────────────┴──────────────────┘
                         │
                         ▼
              ┌─────────────────────┐
              │  Ranked Results    │
              │  (semantic + exact) │
              └─────────────────────┘
```

---

See individual component files for detailed documentation.
