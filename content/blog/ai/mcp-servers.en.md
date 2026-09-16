---
author: ["Potato Energy Team", "ponfertato"]
categories: ["gpt", "ai", "automation"]
date: "2026-09-16T18:10:00+03:00"
description: "MCP (Model Context Protocol) in practice: what it is, which servers to use, how to configure on NixOS and pair with local LLMs."
draft: false
series: ["AI Infrastructure"]
slug: "mcp-servers"
tags: ["mcp", "ai", "automation", "nixos", "ollama", "claude", "cursor"]
title: "MCP Servers: Complete Automation Guide with AI"
---

**Model Context Protocol** is an open standard from Anthropic that gives LLMs a standardized way to connect to external tools and data. Think of it as **USB-C for neural networks**: one protocol, many devices.

**Architecture**:

```
┌─────────────┐         ┌─────────────┐         ┌─────────────┐
│  LLM Client │  ←→    │  MCP Server │  ←→    │   Resource  │
│ (Cursor,    │  stdio/ │ (filesystem,│        │  (files, DB,│
│  Claude,    │  HTTP   │  github...) │        │   API...)   │
│  OpenWebUI) │         │             │         │             │
└─────────────┘         └─────────────┘         └─────────────┘
```

- **Client** - app with LLM (Cursor, Claude Desktop, Open WebUI)
- **Server** - wrapper around a resource (Node.js/Python process)
- **Resource** - what we connect to (DB, API, filesystem)

> 💡 MCP isn't about "smart" models. It's about **a standard way to give models hands**.

---

## 📦 Top MCP servers for real automation

### Official (from Anthropic and partners)

| Server                  | What it does             | When needed                |
| ----------------------- | ------------------------ | -------------------------- |
| **filesystem**          | Read/write/search files  | Working with projects      |
| **github**              | Issues, PRs, repos, code | GitOps automation          |
| **postgres**            | PostgreSQL queries       | Analytics, admin           |
| **sqlite**              | Local SQLite DBs         | Prototyping                |
| **fetch**               | HTTP requests, scraping  | Web data collection        |
| **puppeteer**           | Browser control          | E2E tests, UI automation   |
| **brave-search**        | Search via Brave API     | Current information        |
| **memory**              | Knowledge graph          | Long-term memory           |
| **sequential-thinking** | Reasoning chains         | Complex tasks              |
| **git**                 | Git operations           | Commits, branches, history |
| **slack**               | Slack workspace          | Team automation            |

### Community (often more useful than official)

| Server             | Purpose                 |
| ------------------ | ----------------------- |
| **docker-mcp**     | Container management    |
| **kubernetes-mcp** | K8s clusters            |
| **ssh-mcp**        | Remote servers          |
| **shell-mcp**      | Shell command execution |
| **notion-mcp**     | Notion workspace        |
| **todoist-mcp**    | Task management         |
| **browserbase**    | Cloud headless browser  |
| **exa-search**     | Semantic search         |

---

## 🖥 MCP clients: where it all works

| Client             | Type              | Local LLMs              | Notes                 |
| ------------------ | ----------------- | ----------------------- | --------------------- |
| **Claude Desktop** | Desktop           | ❌ Claude only          | Official, stable      |
| **Cursor**         | IDE               | ✅ Via Ollama/LM Studio | Best DX for code      |
| **Windsurf**       | IDE               | ✅                      | Cursor alternative    |
| **Cline**          | VS Code ext       | ✅                      | Open-source, flexible |
| **Continue**       | VS Code/JetBrains | ✅                      | Cross-platform        |
| **Open WebUI**     | Web               | ✅ Native               | Pipes + Tools         |
| **LM Studio**      | Desktop           | ✅ Native               | Built-in MCP          |
| **Cherry Studio**  | Desktop           | ✅                      | Popular in Asia       |
| **Jan**            | Desktop           | ✅                      | Local-first           |

---

## ⚙️ Setup on NixOS

### Option 1: Via home-manager (recommended)

```nix
# home.nix
{ pkgs, ... }:
{
  home.packages = with pkgs; [
    nodejs_22        # For npx-based servers
    uv               # For Python servers (faster than pip)
    nodePackages.typescript-language-server
  ];

  # Config for Claude Desktop / Cursor
  xdg.configFile."claude/claude_desktop_config.json".text = builtins.toJSON {
    mcpServers = {
      filesystem = {
        command = "npx";
        args = [
          "-y"
          "@modelcontextprotocol/server-filesystem"
          "/home/ponfertato/projects"
          "/home/ponfertato/documents"
        ];
      };
      github = {
        command = "npx";
        args = [ "-y" "@modelcontextprotocol/server-github" ];
        env.GITHUB_PERSONAL_ACCESS_TOKEN = "<TOKEN>";
      };
      postgres = {
        command = "npx";
        args = [
          "-y"
          "@modelcontextprotocol/server-postgres"
          "postgresql://user:pass@localhost:5432/mydb"
        ];
      };
      memory = {
        command = "npx";
        args = [ "-y" "@modelcontextprotocol/server-memory" ];
      };
      fetch = {
        command = "uvx";
        args = [ "mcp-server-fetch" ];
      };
    };
  };
}
```

### Option 2: NixOS module (system level)

```nix
# modules/mcp.nix
{ config, pkgs, lib, ... }:
{
  environment.systemPackages = with pkgs; [
    nodejs_22
    uv
    # If native packages exist:
    # mcp-server-filesystem
    # mcp-server-github
  ];

  # Optional: systemd user services for always-on servers
  systemd.user.services.mcp-filesystem = {
    enable = true;
    description = "MCP Filesystem Server";
    serviceConfig = {
      ExecStart = "${pkgs.nodejs_22}/bin/npx -y @modelcontextprotocol/server-filesystem /home/ponfertato/projects";
      Restart = "always";
    };
    wantedBy = [ "default.target" ];
  };
}
```

### Option 3: Flake with custom MCP packages

```nix
# flake.nix
{
  inputs.nixpkgs.url = "github:NixOS/nixpkgs/nixos-unstable";

  outputs = { self, nixpkgs }: {
    packages.x86_64-linux = let
      pkgs = nixpkgs.legacyPackages.x86_64-linux;
    in {
      mcp-bundle = pkgs.buildEnv {
        name = "mcp-bundle";
        paths = with pkgs; [
          nodejs_22
          uv
          python312
        ];
      };
    };
  };
}
```

---

## 🤖 Integration with Ollama (local LLMs)

MCP is the transport protocol. Ollama is the LLM engine. Together they give a **fully local AI stack with no internet or API keys**.

### Step 1. Run Ollama

```bash
ollama pull qwen2.5-coder:7b  # Best for tool calling on 16 GB RAM
ollama serve
```

### Step 2. Configure client for Ollama

**In Cursor** (`~/.cursor/mcp.json`):

```json
{
  "mcpServers": {
    "filesystem": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-filesystem", "/home"]
    },
    "github": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-github"],
      "env": { "GITHUB_PERSONAL_ACCESS_TOKEN": "..." }
    }
  }
}
```

In Cursor settings: Models → Ollama → `qwen2.5-coder:7b`.

**In Open WebUI** (via pipes, as you already did):
Use Naga/Pollinations/OpenRouter/Cloudflare manifold pipes + `__tools__` parameter.

### Step 3. Test tool calling

```bash
# Test via curl
curl http://localhost:11434/api/chat -d '{
  "model": "qwen2.5-coder:7b",
  "messages": [{"role": "user", "content": "List files in /tmp"}],
  "tools": [...]
}'
```

---

## 🛠 Practical automation scenarios

### Scenario 1: GitOps assistant

**Stack:** `github` + `filesystem` + `git` MCP

- Automatic PR review
- Changelog generation from commits
- Issue migration between repos

### Scenario 2: DevOps helper

**Stack:** `docker` + `ssh` + `postgres` + `shell` MCP

- Container monitoring
- DB migration execution
- Deployment via SSH

### Scenario 3: Research agent

**Stack:** `fetch` + `brave-search` + `memory` + `filesystem`

- Web information collection
- Save to knowledge graph
- Markdown report generation

### Scenario 4: Personal secretary

**Stack:** `notion` + `todoist` + `slack` + `memory`

- Task synchronization
- Slack thread summarization
- Day planning

---

## ⚠️ Your hardware limitations (i5-6500 + 16 GB)

| Component       | What fits                                  | What doesn't    |
| --------------- | ------------------------------------------ | --------------- |
| **LLM**         | 7-8B Q4 (~5 GB RAM)                        | 70B, 32B fp16   |
| **MCP servers** | 5-10 simultaneously (Node.js ~100 MB each) | 50+             |
| **Context**     | 8-16K tokens comfortable                   | 128K+           |
| **Speed**       | 5-10 tok/s on CPU                          | real-time voice |

**Optimizations for 16 GB RAM:**

1. Use **Q4_K_M** quantizations (not fp16)
2. Models: `qwen2.5-coder:7b`, `llama3.1:8b`, `mistral:7b`
3. Don't keep all MCP servers running - launch on demand
4. Limit `num_ctx` in Ollama: `OLLAMA_NUM_CTX=8192`
5. Close GUI clients when not in use (Cursor eats 2+ GB)

---

## Links

- 📘 [Official MCP Specification](https://modelcontextprotocol.io)
- 🐙 [Anthropic MCP Servers](https://github.com/modelcontextprotocol/servers)
- 🗂 [Awesome MCP Servers](https://github.com/punkpeye/awesome-mcp-servers)
- 🦙 [Ollama + MCP](https://github.com/ollama/ollama)
- 🖥 [Cursor MCP Docs](https://docs.cursor.com/context/model-context-protocol)
- 📦 [MCP on NixOS (community)](https://github.com/nix-community/mcp-servers)
