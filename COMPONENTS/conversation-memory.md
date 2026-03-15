# Conversation Memory

Session storage, summarization, and embeddings for conversations.

## Overview

Conversation Memory captures chat history, generates condensed summaries, and makes past conversations searchable.

## How It Works

### Storage

Messages are stored in two tables:

1. **conversation_messages** — Raw messages with:
   - `conv_id` — conversation identifier
   - `role` — user/assistant/system
   - `content` — message text
   - `timestamp` — when sent

2. **conversation_summaries** — DAG-structured summaries with:
   - `conv_id` — conversation identifier
   - `summary` — condensed text
   - `embedding` — vector representation
   - `created_at` — when summarized

### Summarization

Summaries are DAG-based (Directed Acyclic Graph), capturing:
- Key decisions
- Important facts
- Action items
- Topics discussed

This allows agents to:
- **Search** summaries for relevant context
- **Expand** summaries back to original messages when needed

## Usage

### Store Messages

```bash
conv-memory store <conv_id> < messages.json
```

Example `messages.json`:
```json
[
  {"role": "user", "content": "What's the weather?"},
  {"role": "assistant", "content": "It's sunny today!"},
  {"role": "user", "content": "Great, let's go to the park"}
]
```

### Summarize

```bash
conv-memory summarize <conv_id>
```

Creates a DAG summary of the conversation.

### Search

```bash
conv-memory search "what did we decide about the design"
```

Searches across all conversation summaries.

### Expand

```bash
conv-memory expand <conv_id>
```

Expands a summary back to the original messages.

### Stats

```bash
conv-memory stats
```

Shows memory statistics.

## Integration with agent-memory

Conversation summaries are automatically included in `agent-memory query` results unless `--no-conversations` is specified.

```bash
# Include conversations (default)
agent-memory query "project meeting"

# Exclude conversations
agent-memory query "project meeting" --no-conversations
```

## Retention

The system is append-only for messages. Consider:
- Archiving old conversations
- Pruning summaries for very old conversations
- Exporting for long-term storage

---

See also:
- [conv-memory-cli.py](../scripts/conv-memory-cli.py)
- [conversation_memory.py](../scripts/conversation_memory.py)
