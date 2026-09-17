---
author: ["Potato Energy Team", "ponfertato"]
categories: ["gpt", "ai", "tutorial"]
date: "2026-03-16T15:25:00+03:00"
description: "Practical guide to GPT4Free: installation, configuration, and usage of free GPT-4/5 alternatives via Python and TypeScript."
draft: false
series: ["GPT4Free"]
slug: "gpt4free"
tags: ["git", "gpt", "python", "typescript", "api", "docker"]
title: "GPT4Free: Access to GPT-5, DeepSeek, and Gemini"
---

GPT4Free (g4f) is a tool that provides access to GPT-4/5, Claude, Gemini, and DeepSeek models via reverse engineering of public APIs.

> ⚠️ **Note**: for educational and testing purposes only. May violate some services' terms.

---

## Installation

### Requirements

- Computer with internet access
- [Python 3.10+](https://python.org) (check "Add to PATH" during installation)

### Install command

```bash
pip install -U g4f[all]
```

---

## Launch

### Option 1: Web UI

```bash
python -m g4f.cli gui --port 8080
```

Interface: `http://localhost:8080/chat/`

### Option 2: Local API

```bash
python -m g4f --port 1337
```

After launch, applications supporting the OpenAI API can connect to it.

---

## First Script

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

Run:

```bash
python test.py
```

---

## Image Generation

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

## Available Models (March 2026)

| Model            | Status          | Purpose                 |
| ---------------- | --------------- | ----------------------- |
| `gpt-4o-mini`    | ✅ Stable       | Fast responses, chat    |
| `gpt-4o`         | ✅ Stable       | Complex tasks           |
| `deepseek-v3`    | ✅ Stable       | Code, math, logic       |
| `gemini-2.5-pro` | ⚠️ Intermittent | Multimodal tasks        |
| `llama-3.3-70b`  | ✅ Stable       | Open-source alternative |
| `gpt-5`          | 🔶 Experimental | May not work            |

> 💡 The list changes frequently. Get the current list via the web interface or request:
> `GET http://localhost:8080/backend-api/models`

---

## Connecting to OpenAI-compatible Apps

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

Compatible with LibreChat, Flowise, AnythingLLM, and others.

---

## Diagnostics

```bash
# Update library
pip install -U g4f

# Install errors on Windows
pip install --upgrade pip setuptools wheel

# Model not responding
# → Try a different model
# → Enable VPN
# → Wait 10-30 seconds (some providers are slow)

# When using Docker, add more shared memory for the browser:
docker run -p 8080:8080 --shm-size="2g" hlohaus789/g4f:latest
```

---

## Docker

```bash
docker run -p 8080:8080 --shm-size="2g" hlohaus789/g4f:latest
```

- Web UI: `http://localhost:8080`
- API: `http://localhost:8080/v1`

---

## Links

- [Official repository](https://github.com/xtekky/gpt4free)
- [Docker image](https://hub.docker.com/r/hlohaus789/g4f)
- [Documentation](https://g4f.dev)
- [Telegram channel](https://t.me/g4f_channel)
