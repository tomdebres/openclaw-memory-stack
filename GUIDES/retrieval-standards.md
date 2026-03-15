# Retrieval Standards

When and how agents should query memory.

## Overview

Memory is a tool — knowing when to use it is as important as knowing how.

## When to Query Memory

### Do Query When:

1. **User references past context**
   - "What did we decide about X?"
   - "Remember when we discussed Y?"
   - "Earlier you mentioned Z"

2. **Answer depends on indexed knowledge**
   - Repo structure / code patterns
   - Vault notes / documentation
   - Past conversation summaries

3. **Task spans multiple areas**
   - Broad questions that need routing hints
   - Cross-referencing multiple files
   - Finding likely files or terms

4. **About to ask the user something**
   - Check if memory already has the answer
   - Verify preferences before assuming

### Don't Query When:

1. **Exact file is already open**
   - You're looking at the authoritative source
   - The user just pasted the content

2. **Direct inspection is faster**
   - One specific file, no search needed
   - Exact command or code edit in one place

3. **Task is purely local**
   - Math calculations
   - Pure code generation
   - Formatting/transformation

## How Many Results

| Scenario | Recommended Limit |
|----------|-------------------|
| Quick check | `--limit 3` |
| Broad topic | `--limit 5` |
| Exploration | `--limit 10` |

Start small. Widen only if first pass is thin.

## Filtering Tips

Use filters to improve precision:

```bash
# Search only in a repo
agent-memory query "auth" --source-type repo

# Search only in notes
agent-memory query "meeting notes" --source-type vault

# Search specific domain
agent-memory query "api" --domain backend

# Search specific path
agent-memory query "config" --path-prefix scripts/
```

## Trust Policy

### Memory Is Good For:
- Recalling candidate files
- Finding likely terms, headings, path prefixes
- Recovering past decisions/preferences
- Routing hints

### Memory Is NOT Good For:
- Quoting exact details
- Making edits
- Relying on exact numbers, commands, code
- Final truth (when source might be stale)

### The Rule

> **Memory is for locating and reminding. Files are for verifying.**

Always directly inspect the authoritative source when it matters.

## Example Workflow

1. **Query**: `agent-memory query "trading strategy"`
2. **Results**: "Check scripts/trading_commands.py line 42"
3. **Verify**: Open the file, read the actual code
4. **Use**: Apply the verified information

## Integration with Agent Identity

Each agent (Concierge, Builder, etc.) should:
1. Query memory at session start
2. Check relevant context before answering
3. Update memory after significant events

See your agent's `IDENTITY.md` for specific guidance.

## Metrics

Monitor retrieval usage:
- Query frequency
- Result quality (did it help?)
- False positive rate

This helps tune:
- Chunk sizes
- Indexing policy
- Metadata tags
