# Discord Alerts

![Discord Alert Example](../docs/discord-alert-example.jpg)

Set up Discord notifications for memory stack health alerts.

## Recommended framing

For most non-trivial deployments, treat Discord **bot delivery as the primary option** and **webhooks as the lightweight fallback**.

Why:

- bots support richer formatting and routing
- bots are easier to extend later
- webhooks are great for a single simple channel post, but less flexible

That said, both are valid.

## Overview

| Method | Recommendation | Best For |
|--------|----------------|----------|
| **Discord Bot** | Preferred default | Rich embeds, channel routing, DMs, future extension |
| **Discord Webhook** | Optional fallback | Simple one-channel notifications with minimal setup |

## Method 1: Discord Bot (Preferred)

Using a bot token gives you full control over Discord's API, including rich embeds, multiple channels, and direct messages.

### Prerequisites

- A Discord server where you can create/invite a bot
- `discord.py` installed: `pip install discord.py`

### Creating a Bot

1. Go to [Discord Developer Portal](https://discord.com/developers/applications)
2. Create an application (or use an existing one)
3. Open **Bot** → **Reset Token** → copy the token
4. Under **OAuth2** → **URL Generator**:
   - Scopes: `bot`
   - Permissions: `Send Messages`, `Embed Links`
5. Invite the bot to the target server/channel

### Configuring

```bash
export DISCORD_BOT_TOKEN="your-bot-token-here"
export DISCORD_CHANNEL_ID="123456789012345678"
export DISCORD_USE_BOT="1"
```

To get the channel ID:

- enable Developer Mode in Discord
- right-click the channel
- choose **Copy Channel ID**

### Example send logic

```python
import discord
import os
import asyncio

async def send_alert(content: str, embed: discord.Embed = None):
    token = os.getenv("DISCORD_BOT_TOKEN")
    channel_id = int(os.getenv("DISCORD_CHANNEL_ID"))

    intents = discord.Intents.default()
    client = discord.Client(intents=intents)

    await client.login(token)
    channel = await client.fetch_channel(channel_id)
    await channel.send(content=content, embed=embed)
    await client.close()

async def send_health_alert(lm_status: str, pg_status: str):
    embed = discord.Embed(
        title="🔴 OpenClaw Memory Stack Alert",
        color=discord.Color.red(),
    )
    embed.add_field(name="Embeddings", value=lm_status, inline=True)
    embed.add_field(name="PostgreSQL/pgvector", value=pg_status, inline=True)
    await send_alert("**Memory Stack Health Check Failed**", embed)

asyncio.run(send_health_alert("UNREACHABLE", "OK"))
```

### Why bot-first

- rich embeds and clearer status output
- multiple channels if you later separate warning vs failure paths
- easier extension to DMs or richer workflows
- avoids scattering webhook URLs to every place that might post

### Bot security

- never commit the bot token
- store it in `.env` or your secret manager
- rotate it if exposed
- keep bot permissions minimal

## Method 2: Discord Webhook (Optional fallback)

Webhooks are the fast path when you only need a simple alert into one channel.

### Configuring

```bash
export DISCORD_MEMORY_WEBHOOK_URL="https://discord.com/api/webhooks/..."
```

### Example send logic

```bash
curl -X POST "$DISCORD_MEMORY_WEBHOOK_URL" \
  -H "Content-Type: application/json" \
  -d '{"content": "🔴 Memory Stack Alert: embeddings unreachable"}'
```

With an embed:

```bash
curl -X POST "$DISCORD_MEMORY_WEBHOOK_URL" \
  -H "Content-Type: application/json" \
  -d '{
    "embeds": [{
      "title": "🔴 OpenClaw Memory Stack Alert",
      "color": 15158332,
      "fields": [
        {"name": "Embeddings", "value": "UNREACHABLE", "inline": true},
        {"name": "PostgreSQL/pgvector", "value": "OK", "inline": true}
      ]
    }]
  }'
```

## Alert behavior

Good defaults:

- post on failures or degraded health
- avoid hourly "everything is fine" noise unless you explicitly want heartbeats
- include freshness/drift context when relevant

Useful alert types:

- embedding provider unreachable
- PostgreSQL/pgvector unreachable
- document index stale
- conversation sync stale
- low embedding coverage
- drift between raw sessions and indexed/summarised state

## Scheduling examples

This stack is scheduler-agnostic.

### LaunchAgent / supervisor style

Run the alert script on a recurring schedule with the required environment injected by your launcher.

### Cron example (bot mode)

```bash
0 * * * * cd /path/to/openclaw-memory-stack && \
  DISCORD_BOT_TOKEN="your-token" \
  DISCORD_CHANNEL_ID="123456789" \
  DISCORD_USE_BOT="1" \
  python scripts/memory_discord_alert.py >> /var/log/openclaw-memory-alerts.log 2>&1
```

### Cron example (webhook mode)

```bash
0 * * * * cd /path/to/openclaw-memory-stack && \
  DISCORD_MEMORY_WEBHOOK_URL="https://..." \
  python scripts/memory_discord_alert.py >> /var/log/openclaw-memory-alerts.log 2>&1
```

## Choosing between bot and webhook

| Use bot if... | Use webhook if... |
|---------------|-------------------|
| you want the recommended default | you need the quickest possible setup |
| you want richer embeds or future routing | you only post to one channel |
| you may expand alert workflows later | you want minimal moving parts |

## Troubleshooting

### Bot issues

- **invalid token** — token is wrong or was reset
- **cannot message channel** — missing permissions in that channel
- **403/forbidden** — bot role or access is wrong

### Webhook issues

- **invalid webhook URL** — check the full URL
- **webhook not found** — it may have been deleted
- **nothing posted** — verify the environment variable and script path

## Security

- never commit tokens or webhook URLs
- keep secrets in `.env` or a secret manager
- rotate exposed credentials quickly
- prefer least privilege
