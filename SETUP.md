# Setup Guide

This guide walks you through installing and configuring the OpenClaw Memory Stack.

## Prerequisites

- Python 3.10 or higher
- [DuckDB](https://duckdb.org/) (for vector storage)
- At least one embedding service:
  - [LM Studio](https://lmstudio.ai/) (local, free)
  - [Ollama](https://ollama.ai/) (local, free)
  - OpenAI API (cloud, paid)

## Optional: Networking

If your memory stack components run across multiple machines (e.g., embeddings on one host, database on another), you may want a VPN for secure machine-to-machine communication. [Tailscale](https://tailscale.com/) is a common choice — it's optional and not required by the stack itself.

## Step 1: Clone and Install

```bash
git clone https://github.com/tomdebres/openclaw-memory-stack.git
cd openclaw-memory-stack

# Create a virtual environment (recommended)
python3 -m venv .venv
source .venv/bin/activate  # On Windows: .venv\Scripts\activate

# Install dependencies
pip install -r requirements.txt
```

## Step 2: Environment Configuration

Copy the example environment file and fill in your values:

```bash
cp .env.example .env
```

Edit `.env` with your settings:

### Database

```bash
# DuckDB with pgvector extension
DATABASE_URL="duckdb:///openclaw_memory.duckdb"
# Or PostgreSQL with pgvector
DATABASE_URL="postgresql://user:pass@localhost:5432/openclaw"
```

### Embedding Service

Choose one:

```bash
# LM Studio (local, recommended for starters)
LM_STUDIO_URL="http://localhost:1234/v1"
LM_STUDIO_EMBED_MODEL="nomic-embed-text"

# Ollama (local)
OLLAMA_URL="http://localhost:11434"
OLLAMA_EMBED_MODEL="nomic-embed-text"

# OpenAI (cloud)
OPENAI_API_KEY="sk-..."
OPENAI_EMBED_MODEL="text-embedding-3-small"
```

### Optional: Discord Alerts

```bash
# For health monitoring alerts
DISCORD_MEMORY_WEBHOOK_URL="https://discord.com/api/webhooks/..."
```

### Optional: Custom Paths

```bash
# Default paths if not set
OPENCLAW_REPO_MEMORY_PATH="/path/to/your/repo"
OPENCLAW_MEMORY_POLICY_PATH="./memory-layer/index-policy.json"
```

## Step 3: Initialize the Database

```bash
python scripts/db_init.py
```

This creates the necessary tables:
- `memory_documents` — indexed documents metadata
- `memory_chunks` — vector-embedded text chunks
- `memory_index_runs` — indexing run history
- `conversation_summaries` — conversation DAG summaries
- `conversation_messages` — raw conversation messages

## Step 4: Verify Installation

Run a health check to confirm everything is connected:

```bash
python scripts/memory_health.py
```

Expected output:
```
✓ LM Studio reachable
✓ pgvector reachable
✓ Database initialized
Documents: 0 | Chunks: 0
```

## Step 5: Index Your First Documents

### Index a Vault (Obsidian-style notes)

```bash
python scripts/memory_index.py --source-type vault --root /path/to/vault --domain notes
```

### Index a Repository

```bash
python scripts/memory_index.py --source-type repo --root /path/to/repo --domain code
```

Both commands are **incremental** — they'll only re-index files that changed since the last run.

## Step 6: Test Retrieval

```bash
# Search your indexed memory
python scripts/agent-memory.py query "your search term"

# See recent memory events
python scripts/agent-memory.py recent 10

# Check facts
python scripts/agent-memory.py facts
```

## Optional: Set Up Cron Jobs

Add these to your crontab for automated indexing and health monitoring:

```bash
# Index every 15 minutes
*/15 * * * * cd /path/to/openclaw-memory-stack && python scripts/index-memory-repo >> /var/log/openclaw-index.log 2>&1

# Health check every hour (posts to Discord only on failures)
0 * * * * cd /path/to/openclaw-memory-stack && python scripts/memory_discord_alert.py >> /var/log/openclaw-health.log 2>&1
```

## Troubleshooting

### "LM Studio not reachable"

- Make sure LM Studio is running
- Check the URL in your `.env` matches LM Studio's server settings
- Verify the embedding model is downloaded

### "pgvector extension not found"

For DuckDB, the extension loads automatically. For PostgreSQL, ensure pgvector is installed:

```sql
CREATE EXTENSION IF NOT EXISTS vector;
```

### "Permission denied" on scripts

Make scripts executable:

```bash
chmod +x scripts/agent-memory
chmod +x scripts/index-memory-*
```

## Next Steps

- Read [API.md](API.md) for the full CLI reference
- Check [COMPONENTS.md](COMPONENTS.md) to understand each part
- See [GUIDES/](GUIDES/) for specific how-tos

## Upgrading

Pull the latest and reinstall:

```bash
git pull origin main
pip install -r requirements.txt
python scripts/db_init.py  # Run again if schema changed
```
