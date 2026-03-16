---
author: ["Potato Energy Team", "ponfertato"]
categories: ["gpt", "ai", "tutorial"]
date: "2026-03-16T18:00:00+03:00"
description: "Practical Guide to GPT4Free: Installing, Configuring, and using free GPT-4/5 analogues via Python and TypeScript."
draft: false
series: ["GPT4Free"]
slug: "gpt4free"
tags: ["git", "gpt", "python", "typescript", "api", "docker"]
title: "GPT4Free in 2026: A complete guide to free access to GPT-5, DeepSeek and Gemini"
---

GPT4Free (g4f) is a free tool that gives you access to powerful AI models: GPT-4/5, Claude, Gemini, DeepSeek. It works by reverse-engineering public APIs.

> ⚠️ **Note**: For educational and testing purposes only. May violate some services' ToS.

---

## Install in 2 minutes

### Requirements

- Any computer with internet
- [Python 3.10+](https://python.org) (check "Add to PATH" during install)

### One command

```bash
pip install -U g4f[all]
```

Done. Library is ready.

---

## Run it

### Option 1: Web UI (chat in your browser)

```bash
python -m g4f.cli gui --port 8080
```

Open in browser: `http://localhost:8080/chat/`

### Option 2: Developer mode (local API)

```bash
python -m g4f --port 1337
```

Now you can connect any app that supports OpenAI API.

---

## Your first script: 5 lines

Create `test.py`:

```python
from g4f.client import Client

client = Client()

response = client.chat.completions.create(
    model="gpt-4o-mini",
    messages=[{"role": "user", "content": "Why is potato a state of mind?"}]
)

print(response.choices[0].message.content)
```

Run it:

```bash
python test.py
```

---

## Generate images

```python
from g4f.client import Client

client = Client()

img = client.images.generate(
    model="flux",
    prompt="Cyberpunk potato in a neon city",
    response_format="url"
)

print(f"Done: {img.data[0].url}")
```

---

## Working models (March 2026)

| Model            | Status          | Best for                 |
| ---------------- | --------------- | ------------------------ |
| `gpt-4o-mini`    | ✅ Stable       | Fast chat, quick answers |
| `gpt-4o`         | ✅ Stable       | Complex tasks, reasoning |
| `deepseek-v3`    | ✅ Stable       | Code, math, logic        |
| `gemini-2.5-pro` | ⚠️ Intermittent | Multimodal tasks         |
| `llama-3.3-70b`  | ✅ Stable       | Open-source alternative  |
| `gpt-5`          | 🔶 Experimental | May not work             |

> 💡 List changes often. Get the live list via:  
> `GET http://localhost:8080/backend-api/models`

---

## Connect to any OpenAI-compatible app

After running `python -m g4f --port 1337`:

```python
from openai import OpenAI

client = OpenAI(
    base_url="http://localhost:1337/v1",
    api_key="doesnt-matter"  # any value works
)

response = client.chat.completions.create(
    model="gpt-4o-mini",
    messages=[{"role": "user", "content": "Tell me a potato joke"}]
)

print(response.choices[0].message.content)
```

Works with LibreChat, Flowise, AnythingLLM, and more.

---

## Troubleshooting

```bash
# Update the library
pip install -U g4f

# Install errors on Windows
pip install --upgrade pip setuptools wheel

# Model not responding
# → Try a different model
# → Enable VPN
# → Wait 10-30 seconds (some providers are slow)

# Using Docker? Give the browser more shared memory:
docker run -p 8080:8080 --shm-size="2g" hlohaus789/g4f:latest
```

---

## Docker (for servers & advanced users)

```bash
docker run -p 8080:8080 --shm-size="2g" hlohaus789/g4f:latest
```

- Web UI: `http://localhost:8080`
- API: `http://localhost:8080/v1`

---

## Links

- 🐍 [Official repo](https://github.com/xtekky/gpt4free)
- 📦 [Docker image](https://hub.docker.com/r/hlohaus789/g4f)
- 🌐 [Docs](https://g4f.dev)
- 💬 [Telegram channel](https://t.me/g4f_channel)
