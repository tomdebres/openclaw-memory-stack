# Health Monitoring

Automated health checks and Discord alerts for the memory stack.

## Overview

Health Monitoring ensures your memory stack stays operational:
- LM Studio/Ollama reachability
- pgvector connectivity
- Document/chunk counts
- Index freshness

## Components

### memory_health.py

On-demand health checks.

```bash
# Human-readable output
python scripts/memory_health.py

# JSON for bots/cron
python scripts/memory_health.py --json

# Compact Discord text
python scripts/memory_health.py --discord
```

**Checks performed:**
1. Embedding service (LM Studio/Ollama) is reachable
2. pgvector is accessible
3. Database is initialized
4. Document/chunk counts are non-zero
5. Last index run is recent (within `--stale-after-hours`)

### memory_discord_alert.py

Scheduled alerts to Discord.

```bash
# Preview without posting
python scripts/memory_discord_alert.py --dry-run

# Post only on failures
python scripts/memory_discord_alert.py

# Always post heartbeat
python scripts/memory_discord_alert.py --always-post
```

**Alert conditions:**
- Embedding service unreachable
- pgvector connectivity failed
- Database not initialized
- Index is stale (default: 24 hours)
- Zero documents/chunks

## Cron Setup

```bash
# Index every 15 minutes
*/15 * * * * cd /path/to/tomos-memory-stack && python scripts/index-memory-repo >> /var/log/tomos-index.log 2>&1

# Health alert every hour
0 * * * * cd /path/to/tomos-memory-stack && python scripts/memory_discord_alert.py >> /var/log/tomos-health.log 2>&1
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
🟢 TomOS Memory Stack
• LM Studio: OK
• pgvector: OK
• Docs: 142 | Chunks: 1,203
• Last index: 2 hours ago
```

### Failure Alert

```
🔴 TomOS Memory Stack Alert
• LM Studio: UNREACHABLE (connection refused)
• Last index: 26 hours ago (STALE)
```

## Environment Variables

| Variable | Description |
|----------|-------------|
| `LM_STUDIO_URL` | Embedding service URL |
| `DATABASE_URL` | Database connection |
| `DISCORD_MEMORY_WEBHOOK_URL` | Discord webhook |

---

See also:
- [memory_health.py](../scripts/memory_health.py)
- [memory_discord_alert.py](../scripts/memory_discord_alert.py)
