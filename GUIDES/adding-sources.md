# Adding Sources

How to index new data sources beyond vaults and repositories.

## Overview

The memory stack supports multiple source types. This guide shows how to add new ones.

## Existing Source Types

| Type | Description | Command |
|------|-------------|---------|
| `vault` | Obsidian-style markdown notes | `--source-type vault` |
| `repo` | Code repositories | `--source-type repo` |

## Adding a New Source Type

### Step 1: Define in Policy

Edit `memory-layer/index-policy.json`:

```json
{
  "my_source": {
    "allow": ["**/*.myext"],
    "exclude": ["**/test/**"]
  }
}
```

### Step 2: Create Index Command

Option A: Extend `memory_index.py`:

```python
# Add to source_type choices
parser.add_argument('--source-type', choices=['vault', 'repo', 'my_source'])
```

Option B: Create a wrapper script:

```bash
#!/bin/bash
# scripts/index-my-source
exec python scripts/memory_index.py --source-type my_source --root "$1" "$@"
```

### Step 3: Run Indexing

```bash
python scripts/memory_index.py --source-type my_source --root /path/to/data
```

## Indexing External Sources

### From URL

```bash
# Fetch and index
curl -s https://example.com/docs | python scripts/memory_index.py --source-type vault
```

### From API

```bash
# Fetch JSON and store
curl -s api.example.com/data | python scripts/conv-memory store my_api_data
```

### From Database

```bash
# Query and index
python -c "
import duckdb
conn = duckdb.connect('data.duckdb')
for row in conn.execute('SELECT * FROM items'):
    print(row)
" | python scripts/memory_index.py --source-type db
```

## Common Patterns

### Selective Indexing

Index only specific subdirectories:

```bash
python scripts/memory_index.py --source-type repo --root ~/projects --path-prefix "src/"
```

### Domain Tagging

Tag with domain for better retrieval:

```bash
python scripts/memory_index.py --source-type vault --root ~/notes --domain personal
python scripts/memory_index.py --source-type vault --root ~/work --domain work
```

## Best Practices

1. **Start small** — Use `--limit` for smoke tests
2. **Check policy** — Ensure sensitive paths are in `never_index`
3. **Verify embeddings** — Run a test query after indexing
4. **Monitor** — Set up health alerts for new sources

## Troubleshooting

### "No files found"

- Check the `allow` patterns in policy
- Verify the root path exists
- Check `exclude` patterns aren't too broad

### "Embedding failed"

- Verify embedding service is running
- Check model is downloaded
- Review API key (if using OpenAI)

### "Wrong domain"

- Use `--domain` to override
- Check `domain_overrides` in policy
