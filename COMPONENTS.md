# Components

The TomOS Memory Stack is built from four main components:

| Component | Description |
|-----------|-------------|
| [File Memory](COMPONENTS/file-memory.md) | pgvector-backed document indexing |
| [Conversation Memory](COMPONENTS/conversation-memory.md) | Session storage and embeddings |
| [Health Monitoring](COMPONENTS/health-monitoring.md) | Cron jobs and Discord reports |
| [Unified Search](COMPONENTS/unified-search.md) | CLI and API for retrieval |

## Quick Overview

### File Memory

- Incremental indexing of vaults and repositories
- Policy-driven corpus controls (allow/exclude/never-index)
- Metadata: project, domain, path, tags

### Conversation Memory

- Store raw conversation messages
- Generate DAG summaries
- Search and expand summaries

### Health Monitoring

- LM Studio/Ollama reachability checks
- pgvector connectivity
- Index freshness monitoring
- Discord webhook alerts

### Unified Search

- Single CLI entry point: `agent-memory`
- Query, recent, facts, preferences, metrics commands
- Filtering by source-type, domain, path-prefix

---

See individual component files for detailed documentation.
