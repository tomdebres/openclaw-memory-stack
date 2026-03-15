# Health Monitoring

Automated health checks and Discord alerts for the memory stack.

## What It Does

Health Monitoring ensures your memory stack stays operational by continuously checking:
- Embedding service (LM Studio/Ollama) reachability
- pgvector database connectivity
- Document and chunk counts
- Index freshness (when it was last updated)

When problems are detected, Discord alerts notify you immediately.

## How It Works

### health_check.py — On-Demand Checks

Runs all health checks and reports status:

```bash
# Human-readable output
python scripts/memory_health.py

# JSON for bots/cron
python scripts/memory_health.py --json

# Compact Discord text
python scripts/memory_health.py --discord
```

### The Health Checks

| Check | What It Tests | Failure Condition |
|-------|---------------|-------------------|
| **Embedding Service** | LM Studio/Ollama is reachable | Connection refused / timeout |
| **pgvector** | Database is accessible | Connection failed |
| **Database Init** | Tables exist | Missing tables |
| **Document Count** | Files are indexed | Zero documents |
| **Chunk Count** | Chunks exist | Zero chunks |
| **Index Freshness** | Last index run is recent | Beyond stale threshold |

### memory_discord_alert.py — Scheduled Alerts

Posts health status to Discord on a schedule:

```bash
# Preview without posting
python scripts/memory_discord_alert.py --dry-run

# Post only on failures
python scripts/memory_discord_alert.py

# Always post heartbeat
python scripts/memory_discord_alert.py --always-post
```

### Alert Conditions

Alerts are triggered when:
- Embedding service unreachable
- pgvector connectivity failed
- Database not initialized
- Index is stale (default: 24 hours)
- Zero documents or chunks

## Why It Matters

### Problem: Silent Failures

Without monitoring:
- You don't know when embedding service goes down
- Stale indexes give irrelevant results
- Zero documents means nothing is searchable
- Problems go unnoticed until you actually try to use memory

### Solution: Proactive Awareness

Health monitoring gives you:
- **Immediate notification** when things break
- **Historical context** — how long has it been stale?
- **Peace of mind** — the system is checking itself
- **Automation-friendly** — JSON output for cron/alerting tools

### The Cron Setup

```bash
# Index every 15 minutes
*/15 * * * * cd /path/to/openclaw-memory-stack && python scripts/index-memory-repo >> /var/log/openclaw-index.log 2>&1

# Health alert every hour
0 * * * * cd /path/to/openclaw-memory-stack && python scripts/memory_discord_alert.py >> /var/log/openclaw-health.log 2>&1
```

## Output Examples

### Human Output

```
✓ LM Studio reachable (http://localhost:1234)
✓ pgvector reachable
✓ Database initialized (DuckDB)
Documents: 142 | Chunks: 1,203
Last index: 2024-01-15 14:32:00 (2 hours ago)
Status: HEALTHY
```

### Discord Output

```
🟢 OpenClaw Memory Stack
• LM Studio: OK
• pgvector: OK
• Docs: 142 | Chunks: 1,203
• Last index: 2 hours ago
```

### Failure Alert

```
🔴 OpenClaw Memory Stack Alert
• LM Studio: UNREACHABLE (connection refused)
• Last index: 26 hours ago (STALE)
```

## Environment Variables

| Variable | Description |
|----------|-------------|
| `LM_STUDIO_URL` | Embedding service URL (default: http://localhost:1234) |
| `DATABASE_URL` | Database connection (DuckDB path) |
| `DISCORD_MEMORY_WEBHOOK_URL` | Discord webhook for alerts |
| `STALE_HOURS` | Hours after which index is considered stale |

---

## Example: Daily Health Check

```bash
# Add to crontab
0 9 * * * cd ~/openclaw-memory-stack && python scripts/memory_health.py --discord

# You get a morning notification:
# "🟢 Memory Stack healthy - 142 docs, 1,203 chunks indexed"
```

---

See also:
- [memory_health.py](../scripts/memory_health.py)
- [memory_discord_alert.py](../scripts/memory_discord_alert.py)
- [discord-alerts.md](../GUIDES/discord-alerts.md)
