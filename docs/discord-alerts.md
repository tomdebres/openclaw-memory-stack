# Discord Alerts

![Discord Alert Example](../docs/discord-alert-example.jpg)

Set up Discord notifications for memory stack health alerts.

## Overview

When something goes wrong with your memory stack, Discord alerts notify you immediately. Two methods are supported:

| Method | Complexity | Best For |
|--------|------------|----------|
| **Discord Bot** | Higher | Rich formatting, DMs, multiple channels, full API access |
| **Discord Webhook** | Lower | Simple notifications to a single channel |

---

## Method 1: Discord Bot (Recommended)

Using a bot token gives you full control over Discord's API, including rich embeds, multiple channels, and direct messages.

### Prerequisites

- A Discord server where you have permission to create a bot
- The `discord.py` library installed: `pip install discord.py`

### Creating a Bot

1. Go to [Discord Developer Portal](https://discord.com/developers/applications)
2. Create an application (or use existing)
3. Go to **Bot** → **Reset Token** → Copy the token
4. Under **Privileged Intents**, enable:
   - `Server Members Intent` (if tracking users)
5. Go to **OAuth2** → **URL Generator**
   - Scopes: `bot`
   - Permissions: `Send Messages`, `Embed Links`
6. Use the generated URL to invite the bot to your server

### Configuring

```bash
# Required environment variables
export DISCORD_BOT_TOKEN="your-bot-token-here"
export DISCORD_CHANNEL_ID="123456789012345678"  # Channel for alerts

# Optional: Enable bot mode (set to 1 for bot, 0 or unset for webhook)
export DISCORD_USE_BOT="1"
```

To get your `DISCORD_CHANNEL_ID`:
- Enable Developer Mode in Discord (Settings → Advanced → Developer Mode)
- Right-click a channel → "Copy Channel ID"

### Sending Alerts (Bot)

```python
import discord
import os
import asyncio

async def send_alert(content: str, embed: discord.Embed = None):
    """Send alert via Discord bot."""
    token = os.getenv("DISCORD_BOT_TOKEN")
    channel_id = int(os.getenv("DISCORD_CHANNEL_ID"))
    
    intents = discord.Intents.default()
    client = discord.Client(intents=intents)
    
    await client.login(token)
    channel = await client.fetch_channel(channel_id)
    
    if embed:
        await channel.send(content=content, embed=embed)
    else:
        await channel.send(content=content)
    
    await client.close()

# Example health alert
async def send_health_alert(lm_studio_status: str, pgvector_status: str):
    embed = discord.Embed(
        title="🔴 OpenClaw Memory Stack Alert",
        color=discord.Color.red()
    )
    embed.add_field(name="LM Studio", value=lm_studio_status, inline=True)
    embed.add_field(name="pgvector", value=pgvector_status, inline=True)
    
    await send_alert("**Memory Stack Health Check Failed**", embed)

# Run it
asyncio.run(send_health_alert("UNREACHABLE", "OK"))
```

### Bot Advantages

- **Rich embeds** - Colored messages with fields, thumbnails, footers
- **Multiple channels** - Route alerts to different channels based on severity
- **Direct messages** - Send DMs to specific users
- **Reactions** - Add emoji reactions to messages
- **No webhook URL exposure** - Token is server-side only

### Bot Security

- Never commit bot tokens to git
- Store in `.env` (add to `.gitignore`)
- Rotate tokens periodically via Developer Portal
- Set `MFA Required` on bot for extra security

---

## Method 2: Discord Webhook (Simpler)

Webhooks are the quick way to post messages to a channel without setting up a full bot.

### Prerequisites

- A Discord server where you have permission to create webhooks

### Creating a Webhook

1. Open Discord → Server Settings → Integrations
2. Click "Create Webhook" or "New Webhook"
3. Name it (e.g., "OpenClaw Memory")
4. Select the channel
5. Copy the webhook URL

### Configuring

```bash
export DISCORD_MEMORY_WEBHOOK_URL="https://discord.com/api/webhooks/..."
```

Or add to `.env`:

```
DISCORD_MEMORY_WEBHOOK_URL=https://discord.com/api/webhooks/...
```

### Sending Alerts (Webhook)

```bash
# Simple text alert
curl -X POST "https://discord.com/api/webhooks/..." \
  -H "Content-Type: application/json" \
  -d '{"content": "🔴 Memory Stack Alert: LM Studio unreachable"}'

# With embed (richer formatting)
curl -X POST "https://discord.com/api/webhooks/..." \
  -H "Content-Type: application/json" \
  -d '{
    "embeds": [{
      "title": "🔴 OpenClaw Memory Stack Alert",
      "color": 15158332,
      "fields": [
        {"name": "LM Studio", "value": "UNREACHABLE", "inline": true},
        {"name": "pgvector", "value": "OK", "inline": true}
      ]
    }]
  }'
```

### Testing the Connection

```bash
python scripts/memory_discord_alert.py --dry-run
```

This shows what would be posted without sending.

### Send Alerts

```bash
# Post only when unhealthy (recommended)
python scripts/memory_discord_alert.py

# Always post heartbeat
python scripts/memory_discord_alert.py --always-post
```

---

## Alert Types

### Health Failure

```
🔴 OpenClaw Memory Stack Alert
• LM Studio: UNREACHABLE
• pgvector: OK
```

### Stale Index

```
🟡 OpenClaw Memory Stack Warning
• Index is stale (26 hours old)
• Last run: 2024-01-15 14:32:00
```

### Healthy (with --always-post)

```
🟢 OpenClaw Memory Stack
• LM Studio: OK
• pgvector: OK
• Docs: 142 | Chunks: 1,203
```

---

## Cron Setup

### Bot Mode

```bash
# Health check every hour
0 * * * * cd /path/to/tomos-memory-stack && \
  DISCORD_BOT_TOKEN="your-token" \
  DISCORD_CHANNEL_ID="123456789" \
  DISCORD_USE_BOT="1" \
  python scripts/memory_discord_alert.py >> /var/log/openclaw-health.log 2>&1
```

### Webhook Mode (Post Only on Problems)

```bash
# Health check every hour
0 * * * * cd /path/to/tomos-memory-stack && \
  DISCORD_MEMORY_WEBHOOK_URL="https://..." \
  python scripts/memory_discord_alert.py >> /var/log/openclaw-health.log 2>&1
```

### Webhook Mode (With Heartbeat)

```bash
# Health + heartbeat every hour
0 * * * * cd /path/to/tomos-memory-stack && \
  DISCORD_MEMORY_WEBHOOK_URL="https://..." \
  python scripts/memory_discord_alert.py --always-post >> /var/log/openclaw-health.log 2>&1
```

---

## Choosing a Method

| Use Bot If... | Use Webhook If... |
|---------------|-------------------|
| You want rich colored embeds | You need something quick |
| You need DMs to users | You only post to one channel |
| You want to react to your own messages | Simplicity is priority |
| You need programmatic channel selection | You don't want to manage a bot |

---

## Customizing Alerts

Edit `memory_discord_alert.py` to customize:

- Message format
- Thresholds
- Additional checks
- Different channels for different alerts

---

## Troubleshooting

### Bot Issues

- **"Invalid token"** - Check token is correct, hasn't been reset
- **"Missing intents"** - Ensure required intents are enabled in Developer Portal
- **"Cannot message"** - Bot needs "Send Messages" permission in the channel
- **"403 Forbidden"** - Bot role may be too low in server hierarchy

### Webhook Issues

- **"Webhook URL is invalid"** - Ensure you copied the full URL (starts with `https://discord.com/api/webhooks/`)
- **"Webhook not found"** - The webhook was deleted, create a new one
- **"Nothing is posted"** - Check environment variable is set, run with `--dry-run` to debug

### General Issues

- **"Too many messages"** - Remove `--always-post`, increase cron interval
- **Check cron logs** - Look for errors in `/var/log/openclaw-health.log`

---

## Security

- Never commit tokens or webhook URLs to git
- Use environment variables or `.env` (which should be in `.gitignore`)
- Rotate bot tokens periodically via Developer Portal
- For webhooks, create new ones if old ones are compromised
- Set `MFA Required` on bot for extra security