---
author: ["Potato Energy Team", "ponfertato"]
categories: ["ai", "ollama", "guide"]
date: "2026-09-16T18:00:00+03:00"
description: "Ollama: installation, configuration, integrations with Open WebUI and MCP. Model selection for different hardware, performance tips."
draft: false
series: ["AI Tools"]
slug: "ollama"
tags: ["ollama", "llm", "ai", "open-webui", "mcp", "local-ai"]
title: "Ollama: Local LLMs Without Pain"
---

Ollama is a runtime for running large language models locally. One command - and the model is running. No dependency hell, no virtual environments, no configuration headaches.

```
Your machine → Ollama → Model (Llama, Qwen, Mistral...)
```

**Why it matters:**

- Data never leaves your machine (privacy)
- No internet required for inference
- No subscriptions or limits
- Works on CPU (slow, but works)

---

## Installation

### NixOS

```nix
# configuration.nix
services.ollama = {
enable = true;
# Optional: specify models for auto-loading
# models = [ "qwen2.5:7b" ];
};
```

Apply:

```bash
sudo nixos-rebuild switch
```

### Linux (Debian/Ubuntu)

```bash
curl -fsSL https://ollama.com/install.sh | sh
```

### Windows

1. Download installer from [ollama.com](https://ollama.com/download)
2. Install as a regular application
3. Verify in terminal: `ollama --version`

### Verify Installation

```bash
ollama --version
ollama serve   # Start server (if not auto-start)
```

---

## Basic Usage

### Run a Model

```bash
# Interactive chat
ollama run qwen2.5:7b

# Single prompt
ollama run qwen2.5:7b "Write a Python function for sorting"

# With system prompt
ollama run qwen2.5:7b --system "You are a Linux expert" "How to configure firewall?"
```

### Manage Models

```bash
# List installed models
ollama list

# Pull model without running
ollama pull qwen2.5:7b

# Remove model
ollama rm qwen2.5:7b

# Show model info
ollama show qwen2.5:7b
```

### Available Models

| Model            | Size    | Use Case                    |
| ---------------- | ------- | --------------------------- |
| `qwen2.5:7b`     | ~4.5 GB | General, code, multilingual |
| `llama3.2:3b`    | ~2 GB   | Fast, simple tasks          |
| `mistral:7b`     | ~4 GB   | General, fast               |
| `codellama:7b`   | ~4 GB   | Code-specialized            |
| `gemma2:9b`      | ~5.5 GB | Quality responses           |
| `llama3.1:8b`    | ~4.7 GB | General                     |
| `deepseek-r1:8b` | ~5 GB   | Reasoning, logic            |

---

## Configuration

### Launch Parameters

```bash
# Increase context
ollama run qwen2.5:7b --num-ctx 8192

# Temperature (creativity)
ollama run qwen2.5:7b --temperature 0.7

# Max tokens in response
ollama run qwen2.5:7b --num-predict 1024
```

### Environment Variables

```bash
# Set port (default 11434)
OLLAMA_HOST=0.0.0.0:11434 ollama serve

# Set models directory
OLLAMA_MODELS=/path/to/models ollama serve

# CPU thread count
OLLAMA_NUM_PARALLEL=4 ollama serve
```

In NixOS:

```nix
services.ollama = {
enable = true;
host = "0.0.0.0:11434";  # Network access
};
```

### Custom Model (Modelfile)

Create a `Modelfile`:

```
FROM qwen2.5:7b
SYSTEM You are a Linux and DevOps expert. Answer concisely.
PARAMETER temperature 0.3
PARAMETER num_ctx 8192
```

Create model:

```bash
ollama create my-expert -f Modelfile
ollama run my-expert "How to configure docker-compose?"
```

---

## API

### Local API

Ollama serves HTTP API on port 11434:

```bash
# List models
curl http://localhost:11434/api/tags

# Generate
curl http://localhost:11434/api/generate -d '{
"model": "qwen2.5:7b",
"prompt": "Hello"
}'

# Chat (like OpenAI)
curl http://localhost:11434/api/chat -d '{
"model": "qwen2.5:7b",
"messages": [{"role": "user", "content": "Hello"}]
}'
```

### OpenAI-compatible API

Ollama emulates OpenAI API:

```bash
curl http://localhost:11434/v1/chat/completions \
-H "Content-Type: application/json" \
-d '{
"model": "qwen2.5:7b",
"messages": [{"role": "user", "content": "Hello"}]
}'
```

Python usage:

```python
from openai import OpenAI

client = OpenAI(
base_url="http://localhost:11434/v1",
api_key="ollama"  # any text
)

response = client.chat.completions.create(
model="qwen2.5:7b",
messages=[{"role": "user", "content": "Hello"}]
)
print(response.choices[0].message.content)
```

---

## Integrations

### Open WebUI

Open WebUI natively supports Ollama.

1. Start Ollama: `ollama serve`
2. Open Open WebUI
3. Settings → Connections → Add:

```
URL: http://localhost:11434
```

In Docker:

```yaml
# docker-compose.yml
services:
open-webui:
image: ghcr.io/open-webui/open-webui:main
environment:
OLLAMA_BASE_URL: http://host.docker.internal:11434
ports:
  - "3000:8080"
```

### MCP Servers

Ollama can be used as LLM for MCP clients. Example for MCP client:

```json
{
  "model": "qwen2.5:7b",
  "api_base": "http://localhost:11434/v1",
  "api_key": "ollama"
}
```

### Continue.dev

In `~/.continue/config.json`:

```json
{
  "models": [
    {
      "title": "Ollama Qwen",
      "provider": "ollama",
      "model": "qwen2.5:7b",
      "apiBase": "http://localhost:11434"
    }
  ]
}
```

### Other

| Tool            | How to Connect                                                                       |
| --------------- | ------------------------------------------------------------------------------------ |
| **LibreChat**   | In `librechat.yaml` add `endpoints.custom` with `baseURL: http://localhost:11434/v1` |
| **Flowise**     | In node settings specify `baseURL: http://localhost:11434`                           |
| **AnythingLLM** | In connection settings select Ollama                                                 |

---

## Model Selection for Hardware

### Without GPU (CPU only)

| RAM   | Models                                      | Speed        |
| ----- | ------------------------------------------- | ------------ |
| 8 GB  | `llama3.2:3b`, `qwen2.5:3b`                 | ~10-15 tok/s |
| 16 GB | `qwen2.5:7b`, `mistral:7b`, `llama3.1:8b`   | ~5-10 tok/s  |
| 32 GB | `qwen2.5:14b`, `llama3.1:8b` (higher quant) | ~3-5 tok/s   |
| 64 GB | `qwen2.5:32b`, `llama3.1:70b` (Q4)          | ~1-3 tok/s   |

### With GPU

| VRAM  | Models              | Speed        |
| ----- | ------------------- | ------------ |
| 8 GB  | `qwen2.5:7b` (full) | ~30-50 tok/s |
| 12 GB | `qwen2.5:14b`       | ~20-30 tok/s |
| 24 GB | `qwen2.5:32b`       | ~10-15 tok/s |

**Tip for 16 GB without GPU:**

- Use `qwen2.5:7b` with default quantization (Q4)
- Context 4096-8192 tokens
- Temperature 0.3-0.7

---

## Performance Tips

### 1. Quantization

Lower quantization = less memory, worse quality:

```bash
# Example: qwen2.5:7b-instruct-q4_K_M
ollama pull qwen2.5:7b-instruct-q4_K_M
```

| Quant    | Size    | Quality            |
| -------- | ------- | ------------------ |
| `q8_0`   | ~8 GB   | Excellent          |
| `q5_K_M` | ~5.5 GB | Good               |
| `q4_K_M` | ~4.5 GB | Good (recommended) |
| `q3_K_M` | ~3.5 GB | Medium             |
| `q2_K`   | ~2.5 GB | Low                |

### 2. Context

```bash
# Increase context (requires more memory)
ollama run qwen2.5:7b --num-ctx 8192
```

### 3. Thread Count

```bash
# Set thread count (default = all cores)
OLLAMA_NUM_PARALLEL=4 ollama serve
```

---

## Common Issues

| Issue                         | Solution                                             |
| ----------------------------- | ---------------------------------------------------- |
| `ollama: command not found`   | Restart terminal or add to PATH                      |
| Model won't download          | Check internet, disk space                           |
| Slow generation on CPU        | Use smaller model (3b instead of 7b), reduce context |
| `connection refused`          | Check if `ollama serve` is running                   |
| Open WebUI doesn't see models | Check `OLLAMA_BASE_URL`, restart both services       |
| Model "forgets" context       | Increase `--num-ctx`                                 |
| Out of memory                 | Use smaller model or lower quantization              |

---

## Links

- 🦙 [Ollama official site](https://ollama.com)
- 📚 [Documentation](https://github.com/ollama/ollama)
- 🗂️ [Model library](https://ollama.com/library)
- 🌐 [Open WebUI](https://github.com/open-webui/open-webui)
- 🔗 [MCP Protocol](https://modelcontextprotocol.io)
