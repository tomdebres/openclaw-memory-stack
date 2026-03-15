# Security

Secrets management and security best practices.

## Overview

The memory stack handles sensitive data. This guide covers how to keep it secure.

## Never-Index Policy

The most important security measure: **never index secrets**.

### What Must Never Be Indexed

Add these to `memory-layer/index-policy.json`:

```json
{
  "never_index": [
    "**/.env*",
    "**/*.pem",
    "**/*.key",
    "**/*.p12",
    "**/*.pfx",
    "**/secrets/**",
    "**/.secrets/**",
    "**/*token*",
    "**/*password*",
    "**/*credential*",
    "**/*secret*",
    "**/id_rsa**",
    "**/id_ed25519**",
    "**/known_hosts"
  ]
}
```

### Best Practice: Deny First

When in doubt, exclude first. You can always widen later.

## Environment Variables

### Storing Secrets

Never commit secrets to git. Use:

1. **`.env` file** (add to `.gitignore`)
2. **Environment variables** (export in shell/profile)
3. **Secret managers** (1Password, HashiCorp Vault)

### Example .env

```bash
# Safe to commit (or use .env.example)
DATABASE_URL="duckdb:///tomos_memory.duckdb"
LM_STUDIO_URL="http://localhost:1234/v1"

# Secret - keep out of git
OPENAI_API_KEY="sk-..."
DISCORD_WEBHOOK_URL="https://discord.com/api/webhooks/..."
```

### .gitignore Entry

```
.env
.env.local
*.duckdb
*.db
```

## Database Security

### Local (DuckDB)

- Database file lives locally
- No network exposure by default
- Protect file permissions: `chmod 600 *.duckdb`

### Remote (PostgreSQL)

- Use SSL/TLS connections
- Use strong passwords
- Restrict by IP if possible
- Don't expose publicly

## Embedding Service Security

### LM Studio / Ollama (Local)

- Only accessible from localhost by default
- No authentication
- Don't expose to network unless needed

### OpenAI (Cloud)

- API key required
- Use minimal scopes
- Rotate keys periodically
- Monitor usage

## Discord Webhooks

- Don't commit webhook URLs
- Use environment variables
- Rotate periodically
- Create dedicated webhooks per use case

## Access Control

### Filesystem

```bash
# Restrict access
chmod 700 tomos-memory-stack/
chmod 600 .env
```

### Database

```bash
# DuckDB - file permissions
chmod 600 tomos_memory.duckdb
```

## Monitoring

### What to Watch

1. **Failed logins** — Unauthorized access attempts
2. **Unexpected queries** — Unusual retrieval patterns
3. **Index errors** — Failed indexing of sensitive files (shouldn't happen with proper policy)

### Tools

- Use health monitoring for alerts
- Check logs regularly
- Set up audit trails for sensitive operations

## Incident Response

### If Secrets Are Indexed

1. **Immediate**: Stop indexing
2. **Assess**: What was indexed? How long?
3. **Remediate**:
   - Clear affected pgvector tables
   - Rotate exposed secrets
   - Update never_index policy
4. **Verify**: Confirm no traces remain

## Security Checklist

- [ ] `.env` in `.gitignore`
- [ ] `never_index` covers all secret patterns
- [ ] Database file permissions set (600)
- [ ] No API keys committed
- [ ] Webhook URLs in env vars, not code
- [ ] Regular security audits
- [ ] Health monitoring active

## Questions?

If you're unsure about security implications:
1. Pause before acting
2. Ask the owner
3. Default to caution
