# Embeddings

Configure embedding providers: LM Studio, Ollama, or OpenAI.

## Overview

The memory stack needs an embedding service to convert text into vectors for semantic search.

## Options

### LM Studio (Recommended for Starters)

**Pros:** Free, local, privacy-friendly, easy setup
**Cons:** Requires local GPU/CPU

```bash
# Environment
LM_STUDIO_URL="http://localhost:1234"
LM_STUDIO_EMBED_MODEL="nomic-embed-text"
# or use LM_STUDIO_BASE_URL as alternative env var
```

1. Download [LM Studio](https://lmstudio.ai/)
2. Download an embedding model (e.g., "nomic-embed-text")
3. Start the local server (LM Studio → Server → Start Server)
4. Set environment variables

### Ollama

**Pros:** Free, local, good model selection
**Cons:** Requires setup, less GUI-friendly

```bash
# Environment
OLLAMA_URL="http://localhost:11434"
OLLAMA_EMBED_MODEL="nomic-embed-text"
```

1. Install [Ollama](https://ollama.ai/)
2. Pull the model: `ollama pull nomic-embed-text`
3. Start server: `ollama serve`
4. Set environment variables

### OpenAI

**Pros:** No local setup, powerful models
**Cons:** Costs money, data leaves your machine

```bash
# Environment
OPENAI_API_KEY="sk-..."
OPENAI_EMBED_MODEL="text-embedding-3-small"
```

1. Get an API key from [OpenAI](https://platform.openai.com/)
2. Set environment variables

## Model Recommendations

| Model | Provider | Dimensions | Notes |
|-------|----------|------------|-------|
| `nomic-embed-text` | LM Studio / Ollama | 768 | Best free option |
| `text-embedding-3-small` | OpenAI | 1536 | Good quality |
| `text-embedding-3-large` | OpenAI | 3072 | Highest quality |

## Configuration

### Single Provider

Set only the variables for your provider:

```bash
# LM Studio only
LM_STUDIO_URL="http://localhost:1234/v1"
LM_STUDIO_EMBED_MODEL="nomic-embed-text"
```

### Fallback (Coming Soon)

You can configure fallback, but currently the first available provider is used.

### Verifying Setup

```bash
# Test LM Studio
curl http://localhost:1234/v1/models
# Note: LM Studio appends /v1/embeddings automatically when using the SDK

# Test Ollama
curl http://localhost:11434/api/tags

# Test OpenAI
curl https://api.openai.com/v1/models \
  -H "Authorization: Bearer $OPENAI_API_KEY"
```

## Performance

### LM Studio

- CPU: Works but slower
- GPU (Apple Silicon): Fast, efficient
- GPU (NVIDIA): Fast, efficient

### Ollama

- Uses llama.cpp under the hood
- Good Apple Silicon support
- Memory usage depends on model

### OpenAI

- API latency (~500ms typical)
- No local resources needed
- Pay per token

## Switching Providers

1. Stop current provider
2. Start new provider
3. Update `.env` variables
4. Restart any running scripts
5. Re-run health check

## Troubleshooting

### "Connection refused"

- Provider not running
- Wrong URL (check port)

### "Model not found"

- Model not downloaded
- Wrong model name

### "Rate limit"

- OpenAI: Wait and retry
- Local: Check system resources

### "Embedding dimension mismatch"

- Using wrong model
- Database needs reset
