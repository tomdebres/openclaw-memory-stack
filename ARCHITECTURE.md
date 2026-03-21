# Architecture

High-level design and data flow for the OpenClaw Memory Stack.

## Generic stack vs. a concrete deployment

This document describes the **generic stack pattern**.

A real deployment can swap pieces in and out:

- different embedding providers
- different schedulers
- different alert channels
- different storage locations for raw conversations

Where helpful, notes below call out the **current TomOS implementation** as one practical example rather than a universal requirement.

## Overview

```
┌──────────────────────────────────────────────────────────────────┐
│                         Your AI Agent                            │
└──────────────────────────────────────────────────────────────────┘
                                │
                                ▼
┌──────────────────────────────────────────────────────────────────┐
│                    Agent Memory CLI / helpers                    │
│               `agent-memory query | recent | facts`              │
└──────────────────────────────────────────────────────────────────┘
                                │
        ┌───────────────────────┼───────────────────────┬──────────────────────┐
        ▼                       ▼                       ▼                      ▼
┌─────────────────┐  ┌─────────────────────┐  ┌──────────────────────┐  ┌──────────────────┐
│ Local typed     │  │ Document memory     │  │ Conversation memory  │  │ Health / alerts  │
│ memory-core     │  │ PostgreSQL+pgvector │  │ raw + embeds +       │  │ drift / coverage │
│                 │  │                     │  │ summaries             │  │                  │
└─────────────────┘  └─────────────────────┘  └──────────────────────┘  └──────────────────┘
```

## Layer responsibilities

### 1. Local typed memory

Purpose:

- small durable facts
- user/system preferences
- recent notable history
- counters and metrics

Typical shape:

- `preferences.json`
- `facts.json`
- `history.jsonl`
- `metrics.json`

This layer is simple on purpose. It should not depend on embeddings or database connectivity.

### 2. Document memory

Purpose:

- semantic retrieval over repo docs, notes, standards, runbooks, and other indexed files

Recommended implementation:

- scan allowed roots
- apply an index policy (`allow`, `exclude`, `never_index`)
- chunk files by heading/paragraph
- generate embeddings
- store chunks + metadata in PostgreSQL + pgvector

Typical tables:

- `memory_documents`
- `memory_chunks`
- `memory_index_runs`

TomOS note:

- the current TomOS implementation uses PostgreSQL + pgvector for document memory and treats it as the canonical semantic doc layer

### 3. Conversation memory

Purpose:

- restore continuity across agent/user sessions
- surface useful prior discussions
- support summaries/highlights without losing raw history

Recommended implementation pattern:

1. keep **raw sessions/messages** as the durable source
2. maintain **embeddings** for retrieval
3. maintain **summaries/highlights** for compact recall and admin review

This matters because summaries alone are too lossy, while raw sessions alone are too noisy.

TomOS note:

- current TomOS conversation memory uses local raw session files plus a local embedding/summaries layer and treats drift/coverage as health signals

### 4. Health, drift, and coverage

Purpose:

- detect when memory quietly stops being trustworthy

Checks worth running:

- embedding provider reachability
- PostgreSQL/pgvector reachability
- document index freshness
- conversation sync freshness
- source-to-index drift
- embedding coverage
- summary coverage
- dependency/config sanity

This layer is easy to undervalue and painful to skip.

## Retrieval flow

When an agent queries memory:

1. the query hits the memory CLI/helper layer
2. local typed memory is checked for obvious durable context
3. document memory is queried semantically when configured
4. conversation memory is queried for continuity context
5. results are returned in a blended form or filtered by source type
6. the agent verifies exact claims against the underlying source when needed

Trust order should usually be:

1. live source / direct file inspection
2. canonical docs from document memory
3. conversation memory
4. typed history snippets

## Indexing flow

### Document indexing

```text
File → filter by policy → chunk → embed → store in PostgreSQL+pgvector
```

Good defaults:

- incremental indexing
- non-destructive by default
- explicit prune mode for deletions
- dry-run path for verification

### Conversation indexing/sync

```text
Raw session/message source → sync/import → embed → summarize → persist status
```

Good defaults:

- raw sessions remain the durable truth
- deleted/archived sessions are not silently treated as active
- sync is idempotent
- freshness and coverage are visible in health output

## Scheduling

This stack is **scheduler-agnostic**.

You can run recurring jobs with:

- LaunchAgents
- cron
- systemd timers
- another process supervisor

TomOS note:

- on macOS, TomOS commonly prefers **LaunchAgents** for local always-on jobs and keeps cron examples as fallback/reference

Typical recurring jobs:

- document indexing
- conversation sync
- health checks
- alert emission
- optional daily reports

## Alerting

Discord is a common alert destination.

Recommended generic stance:

- **bot-first** when you want richer formatting or routing flexibility
- **webhook** when you want the simplest possible channel post

Either approach should only send noisy heartbeats if you explicitly want them.

## Security boundaries

The index policy should prevent sensitive material from being embedded.

Common exclusions:

- `.env*`
- private keys and certificates
- password/token-bearing files
- `secrets/` and similar directories
- local databases and transient runtime artifacts

See [SECURITY.md](SECURITY.md) for the operational guidance.

## Extensibility

The stack is intentionally modular:

- swap embedding providers without changing the layer model
- add/remove alert channels without changing retrieval
- tune indexing policy without changing typed memory
- evolve conversation summarisation independently of document retrieval

---

See [README.md](README.md) for the practical overview and [docs/discord-alerts.md](docs/discord-alerts.md) for Discord alerting guidance.
