# File Memory

pgvector-backed document indexing with policy-driven corpus controls.

## What It Does

File Memory indexes documents from vaults (Obsidian-style notes) and repositories, making them searchable via semantic similarity. Unlike keyword search, it understands *meaning* — so "meeting notes" will find "calendar sync discussion" even if those exact words don't appear.

## How It Works

### The Indexing Pipeline

1. **Scan** — Walk the root directory, collect all files matching allow rules
2. **Filter** — Apply policy rules (allow/exclude/never-index)
3. **Chunk** — Split files into semantic chunks (headings, paragraphs, code blocks)
4. **Embed** — Convert chunks to vectors via LM Studio/Ollama/OpenAI
5. **Store** — Save vectors to pgvector (DuckDB) with rich metadata

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

### Metadata Schema

Each chunk carries rich metadata for filtering and ranking:

| Field | Description |
|-------|-------------|
| `project` | Stable project tag |
| `domain` | Logical domain (notes, code, automation) |
| `source_type` | vault or repo |
| `path_parts` | Array of path components |
| `path_parent` | Parent directory |
| `path_depth` | Depth in tree |
| `top_level` | Top-level folder |
| `tags` | Derived labels |

### Incremental Indexing

By default, indexers are **incremental**:
- Each file is hashed after indexing
- Only files with changed content hash are re-embedded
- Use `--full` for intentional full rebuilds

## Why It Matters

### Problem: Keyword Search Is Limited

Traditional search requires exact matches:
- "budget" won't find "financial planning"
- "Python script" won't find "automation code"
- You need to know the right keywords to find anything

### Solution: Semantic Understanding

With vector embeddings:
- "What did Tom write about the trading strategy?" finds relevant notes even without those exact words
- "Find my notes on project X" works without remembering exact filenames
- Related concepts surface automatically

### The Database Tables

| Table | Description |
|-------|-------------|
| `memory_documents` | File metadata (path, hash, last modified) |
| `memory_chunks` | Embedded text + vector (1536 dimensions) |
| `memory_index_runs` | Index run history for freshness checks |

### Ranking Algorithm

Search results are ranked by:
1. **Semantic similarity** — Vector distance (cosine similarity)
2. **Exact text match** — Boost for literal query matches
3. **Heading/title boost** — Chunks from titles rank higher
4. **Path preference** — Configurable path-based boosting

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

### Command Options

| Option | Description |
|--------|-------------|
| `--source-type` | `vault` or `repo` |
| `--root` | Root directory to scan |
| `--domain` | Logical domain tag |
| `--policy` | Path to policy JSON |
| `--full` | Force full rebuild |
| `--limit` | Limit files for smoke test |
| `--json` | JSON output |

---

## Example

```bash
# Index your Obsidian vault
python scripts/memory_index.py --source-type vault --root ~/notes --domain personal

# Now search semantically
agent-memory query "what did I write about my health goals"
# Returns: "nutrition-plan.md" (contains meal planning, related to health)
```

---

See also:
- [memory_index.py](../scripts/memory_index.py)
- [index-memory-*](../scripts/) wrappers
- [EMBEDDINGS.md](../EMBEDDINGS.md) for embedding service setup
