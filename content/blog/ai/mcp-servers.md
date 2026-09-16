---
author: ["Potato Energy Team", "ponfertato"]
categories: ["gpt", "ai", "automation"]
date: "2026-09-16T19:10:00+03:00"
description: "MCP (Model Context Protocol) на практике: что это, какие серверы использовать, как настроить на NixOS и подружить с локальными LLM."
draft: false
series: ["Infrastructure"]
slug: "mcp-servers"
tags: ["mcp", "ai", "automation", "nixos", "ollama", "claude", "cursor"]
title: "MCP-серверы: полный гайд по автоматизации с AI"
---

**Model Context Protocol** - открытый стандарт от Anthropic, который даёт LLM стандартизированный способ подключаться к внешним инструментам и данным. Думай об этом как об **USB-C для нейросетей**: один протокол - множество устройств.

**Архитектура**:

```
┌─────────────┐         ┌─────────────┐         ┌─────────────┐
│  LLM Client │  ←→    │  MCP Server │  ←→    │   Resource  │
│ (Cursor,    │  stdio/ │ (filesystem,│        │  (files, DB,│
│  Claude,    │  HTTP   │  github...) │        │   API...)   │
│  OpenWebUI) │         │             │         │             │
└─────────────┘         └─────────────┘         └─────────────┘
```

- **Клиент** - приложение с LLM (Cursor, Claude Desktop, Open WebUI)
- **Сервер** - обёртка вокруг ресурса (Node.js/Python-процесс)
- **Ресурс** - то, к чему подключаемся (БД, API, файловая система)

> 💡 MCP не про "умную" модель. Он про **стандартный способ дать модели руки**.

---

## 📦 Топ MCP-серверов для реальной автоматизации

### Официальные (от Anthropic и партнёров)

| Сервер                  | Что делает                    | Когда нужен                  |
| ----------------------- | ----------------------------- | ---------------------------- |
| **filesystem**          | Чтение/запись/поиск файлов    | Работа с проектами           |
| **github**              | Issues, PR, репозитории, код  | GitOps автоматизация         |
| **postgres**            | Запросы к PostgreSQL          | Аналитика, администрирование |
| **sqlite**              | Локальные SQLite БД           | Прототипирование             |
| **fetch**               | HTTP-запросы, скрейпинг       | Сбор данных из веба          |
| **puppeteer**           | Управление браузером          | E2E-тесты, автоматизация UI  |
| **brave-search**        | Поиск через Brave API         | Актуальная информация        |
| **memory**              | Граф знаний (knowledge graph) | Долговременная память        |
| **sequential-thinking** | Цепочки рассуждений           | Сложные задачи               |
| **git**                 | Git-операции                  | Коммиты, ветки, история      |
| **slack**               | Slack workspace               | Командная автоматизация      |

### Community (часто полезнее официальных)

| Сервер             | Назначение                |
| ------------------ | ------------------------- |
| **docker-mcp**     | Управление контейнерами   |
| **kubernetes-mcp** | K8s-кластеры              |
| **ssh-mcp**        | Удалённые серверы         |
| **shell-mcp**      | Выполнение shell-команд   |
| **notion-mcp**     | Notion workspace          |
| **todoist-mcp**    | Task-менеджмент           |
| **browserbase**    | Headless-браузер в облаке |
| **exa-search**     | Семантический поиск       |

---

## 🖥 MCP-клиенты: где это всё работает

| Клиент             | Тип               | Локальные LLM             | Примечание              |
| ------------------ | ----------------- | ------------------------- | ----------------------- |
| **Claude Desktop** | Desktop           | ❌ Только Claude          | Официальный, стабильный |
| **Cursor**         | IDE               | ✅ Через Ollama/LM Studio | Лучший DX для кода      |
| **Windsurf**       | IDE               | ✅                        | Альтернатива Cursor     |
| **Cline**          | VS Code ext       | ✅                        | Open-source, гибкий     |
| **Continue**       | VS Code/JetBrains | ✅                        | Мультиплатформенный     |
| **Open WebUI**     | Web               | ✅ Native                 | Pipes + Tools           |
| **LM Studio**      | Desktop           | ✅ Native                 | Встроенный MCP          |
| **Cherry Studio**  | Desktop           | ✅                        | Популярный в Азии       |
| **Jan**            | Desktop           | ✅                        | Локальный-first         |

---

## ⚙️ Настройка на NixOS

### Вариант 1: Через home-manager (рекомендуется)

```nix
# home.nix
{ pkgs, ... }:
{
  home.packages = with pkgs; [
    nodejs_22        # Для npx-based серверов
    uv               # Для Python-серверов (быстрее pip)
    nodePackages.typescript-language-server
  ];

  # Конфиг для Claude Desktop / Cursor
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

### Вариант 2: NixOS module (системный уровень)

```nix
# modules/mcp.nix
{ config, pkgs, lib, ... }:
{
  environment.systemPackages = with pkgs; [
    nodejs_22
    uv
    # Если есть нативные пакеты:
    # mcp-server-filesystem
    # mcp-server-github
  ];

  # Опционально: systemd user-сервисы для постоянно работающих серверов
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

### Вариант 3: Flake с custom MCP-пакетами

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

## 🤖 Интеграция с Ollama (локальные LLM)

MCP - протокол транспорта. Ollama - движок LLM. Вместе они дают **полностью локальный AI-стек без интернета и API-ключей**.

### Шаг 1. Запустить Ollama

```bash
ollama pull qwen2.5-coder:7b  # Лучший для tool calling на 16 GB RAM
ollama serve
```

### Шаг 2. Настроить клиент на Ollama

**В Cursor** (`~/.cursor/mcp.json`):

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

В настройках Cursor: Models → Ollama → `qwen2.5-coder:7b`.

**В Open WebUI** (через pipes, как ты уже делал):
Используй Naga/Pollinations/OpenRouter/Cloudflare manifold-пайпы + `__tools__` параметр.

### Шаг 3. Проверить tool calling

```bash
# Тест через curl
curl http://localhost:11434/api/chat -d '{
  "model": "qwen2.5-coder:7b",
  "messages": [{"role": "user", "content": "List files in /tmp"}],
  "tools": [...]
}'
```

---

## 🛠 Практические сценарии автоматизации

### Сценарий 1: GitOps-ассистент

**Стек:** `github` + `filesystem` + `git` MCP

- Автоматический ревью PR
- Генерация changelog из коммитов
- Миграция issues между репозиториями

### Сценарий 2: DevOps-помощник

**Стек:** `docker` + `ssh` + `postgres` + `shell` MCP

- Мониторинг контейнеров
- Выполнение миграций БД
- Деплой через SSH

### Сценарий 3: Исследовательский агент

**Стек:** `fetch` + `brave-search` + `memory` + `filesystem`

- Сбор информации из веба
- Сохранение в knowledge graph
- Генерация отчётов в Markdown

### Сценарий 4: Личный секретарь

**Стек:** `notion` + `todoist` + `slack` + `memory`

- Синхронизация задач
- Саммаризация Slack-тредов
- Планирование дня

---

## ⚠️ Ограничения твоего железа (i5-6500 + 16 GB)

| Компонент       | Что влезет                                 | Что не влезет   |
| --------------- | ------------------------------------------ | --------------- |
| **LLM**         | 7-8B Q4 (~5 GB RAM)                        | 70B, 32B fp16   |
| **MCP-серверы** | 5-10 одновременно (Node.js ~100 MB каждый) | 50+             |
| **Контекст**    | 8-16K токенов комфортно                    | 128K+           |
| **Скорость**    | 5-10 tok/s на CPU                          | real-time voice |

**Оптимизации для 16 GB RAM:**

1. Используй **Q4_K_M** квантизации (не fp16)
2. Модели: `qwen2.5-coder:7b`, `llama3.1:8b`, `mistral:7b`
3. Не оставляй все серверы MCP постоянно включенными - запускайте их по мере необходимости
4. Ограничь `num_ctx` в Ollama: `OLLAMA_NUM_CTX=8192`
5. Выключи GUI-клиенты когда не используешь (Cursor жрёт 2+ GB)

---

## Ссылки

- 📘 [Официальная спецификация MCP](https://modelcontextprotocol.io)
- 🐙 [Anthropic MCP Servers](https://github.com/modelcontextprotocol/servers)
- 🗂 [Awesome MCP Servers](https://github.com/punkpeye/awesome-mcp-servers)
- 🦙 [Ollama + MCP](https://github.com/ollama/ollama)
- 🖥 [Cursor MCP Docs](https://docs.cursor.com/context/model-context-protocol)
- 📦 [MCP на NixOS (community)](https://github.com/nix-community/mcp-servers)
