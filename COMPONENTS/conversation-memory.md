# Conversation Memory

Session storage, summarization, and embeddings for conversations.

## What It Does

Conversation Memory captures chat history in full, generates condensed DAG-structured summaries, and makes past conversations searchable via semantic similarity. It solves the core problem: **agents forget everything between sessions**.

## How It Works

### Storage Architecture

Messages are stored in two complementary tables:

#### 1. conversation_messages — Full History

| Field | Description |
|-------|-------------|
| `conv_id` | Unique conversation identifier |
| `role` | user, assistant, or system |
| `content` | Raw message text |
| `timestamp` | When the message was sent |

This table stores **everything**. No information is lost.

#### 2. conversation_summaries — DAG Summaries

| Field | Description |
|-------|-------------|
| `conv_id` | Conversation identifier |
| `summary` | Condensed DAG-structured text |
| `embedding` | Vector representation for search |
| `created_at` | When the summary was generated |

### DAG Structured Summaries

The summaries aren't flat — they're Directed Acyclic Graphs capturing:
- **Key decisions** — "Decided to use PostgreSQL instead of MongoDB"
- **Important facts** — "Tom prefers morning meetings"
- **Action items** — "Need to review PR #42"
- **Topics discussed** — "Project planning, budget review"

This structure allows agents to:
- **Search** summaries for relevant context
- **Expand** summaries back to original messages when needed

### The Search Flow

```
User asks: "What did we decide about the API design?"
     │
     ▼
1. Embed the query
     │
     ▼
2. Search conversation_summaries (vector similarity)
     │
     ▼
3. Return matching summaries with relevance scores
     │
     ▼
4. Agent can "expand" any summary to see full messages
```

## Why It Matters

### Problem: Original OpenClaw Used Summarization

The original memory system only stored **summaries**:
- Summaries compress meaning — specific examples, exact phrasing, nuances disappear
- Once summarized, the original messages were discarded
- You couldn't go back to see what was actually said

### Solution: Full Storage + Semantic Search

This system stores **everything**:
- Full messages preserved in `conversation_messages`
- DAG summaries for quick overview
- Vector embeddings enable semantic search
- **Expandability** means you can always get the details back

### Key Benefits

| Benefit | Description |
|---------|-------------|
| **No Amnesia** | Agents remember previous sessions |
| **Full Context** | Original messages preserved, not just summaries |
| **Semantic Search** | "What did we decide about X?" finds relevant past conversations |
| **Expandable** | Summaries can expand back to original messages |
| **Auto-Included** | Conversations searched by default in `agent-memory query` |

### Retention Model

The system is **append-only** for messages:
- Messages are never deleted automatically
- Consider archiving very old conversations
- Export for long-term storage if needed

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

---

## Example

```bash
# Today: Agent has a long conversation with user about project planning
# User mentions: "Remember I prefer morning meetings"

# Next week: User asks "When should we schedule the sync?"
# Agent queries memory -> finds summary: "Tom prefers morning meetings"
# Agent responds: "Based on your preference for morning meetings, how about 10am?"
```

---

See also:
- [conv-memory-cli.py](../scripts/conv-memory-cli.py)
- [conversation_memory.py](../scripts/conversation_memory.py)
