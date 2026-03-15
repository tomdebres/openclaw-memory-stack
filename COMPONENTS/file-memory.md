# File Memory

pgvector-backed document indexing with policy-driven corpus controls.

## Overview

File Memory indexes documents from vaults (Obsidian-style notes) and repositories, making them searchable via semantic similarity.

## How It Works

### Indexing Pipeline

1. **Scan** — Walk the root directory, collect all files
2. **Filter** — Apply policy rules (allow/exclude/never-index)
3. **Chunk** — Split files into semantic chunks (headings, paragraphs)
4. **Embed** — Convert chunks to vectors via LM Studio/Ollama/OpenAI
5. **Store** — Save to pgvector with metadata

### Policy-Driven Corpus Control

The index policy lives in `memory-layer/index-policy.json`:

```json
{
  "vault": {
    "allow": ["**/*.md", "**/*.txt"],
    "exclude": ["**/node_modules/**", "**/.git/**"]
  },
  "repo": {
    "allow": ["**/*.py", "**/*.js", "**/*.md"],
    "exclude": ["**/dist/**", "**/build/**", "**/__pycache__/**"]
  },
  "never_index": [
    "**/.env*",
    "**/*.pem",
    "**/*.key",
    "**/secrets/**",
    "**/.secrets/**"
  ],
  "domain_overrides": {
    "scripts/": "automation"
  }
}
```

### Metadata

Each chunk carries:
- `project` — stable project tag
- `domain` — logical domain (notes, code, automation)
- `source_type` — vault or repo
- `path_parts`, `path_parent`, `path_depth`, `top_level` — path hierarchy
- `tags` — derived labels

## Usage

### Index a Vault

```bash
python scripts/memory_index.py --source-type vault --root ~/notes --domain notes
```

### Index a Repository

```bash
python scripts/memory_index.py --source-type repo --root ~/projects/myapp --domain myapp
```

### Convenience Wrappers

```bash
bash scripts/index-memory-vault
bash scripts/index-memory-repo
```

### Options

| Option | Description |
|--------|-------------|
| `--source-type` | `vault` or `repo` |
| `--root` | Root directory to scan |
| `--domain` | Logical domain tag |
| `--policy` | Path to policy JSON |
| `--full` | Force full rebuild |
| `--limit` | Limit files for smoke test |
| `--json` | JSON output |

## Incremental Indexing

By default, indexers are **incremental**:

- Only files with changed content hash are re-embedded
- Use `--full` for intentional full rebuilds

## What's Stored

| Table | Description |
|-------|-------------|
| `memory_documents` | File metadata (path, hash, last modified) |
| `memory_chunks` | Embedded text + vector |
| `memory_index_runs` | Index run history |

## Ranking

Search results are ranked by:
1. Semantic similarity (vector distance)
2. Exact text match boost
3. Heading/title boost
4. Path preference boost (configurable)

---

See also:
- [memory_index.py](../scripts/memory_index.py)
- [index-memory-*](../scripts/) wrappers
