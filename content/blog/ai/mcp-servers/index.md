---
author: ["Potato Energy Team", "ponfertato"]
categories: ["gpt", "ai", "automation"]
date: "2026-09-16T19:10:00+03:00"
description: "MCP (Model Context Protocol) на практике: что это, какие серверы использовать, как настроить на NixOS и подключить к локальным LLM."
draft: false
series: ["Infrastructure"]
slug: "mcp-servers"
tags: ["mcp", "ai", "automation", "nixos", "ollama", "claude", "cursor"]
title: "MCP-серверы: руководство по автоматизации с AI"
---

**Model Context Protocol** - открытый стандарт от Anthropic, определяющий способ подключения LLM к внешним инструментам и данным. Один протокол - множество источников.

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
- **Ресурс** - источник данных или действий (БД, API, файловая система)

> 💡 MCP определяет стандартный способ подключения модели к внешним инструментам. Модель не становится "умнее" - она получает доступ к данным и действиям.

---

## Топ MCP-серверов

### Официальные (от Anthropic и партнёров)

| Сервер                  | Назначение                    | Когда нужен                  |
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

### Community

| Сервер             | Назначение                |
| ------------------ | ------------------------- |
| **docker-mcp**     | Управление контейнерами   |
| **kubernetes-mcp** | K8s-кластеры              |
| **ssh-mcp**        | Удалённые серверы         |
| **shell-mcp**      | Выполнение shell-команд   |
| **notion-mcp**     | Notion workspace          |
| **todoist-mcp**    | Управление задачами       |
| **browserbase**    | Headless-браузер в облаке |
| **exa-search**     | Семантический поиск       |

---

## MCP-клиенты

| Клиент             | Тип               | Локальные LLM             | Примечание                 |
| ------------------ | ----------------- | ------------------------- | -------------------------- |
| **Claude Desktop** | Desktop           | ❌ Только Claude          | Официальный, стабильный    |
| **Cursor**         | IDE               | ✅ Через Ollama/LM Studio | Оптимальная работа с кодом |
| **Windsurf**       | IDE               | ✅                        | Альтернатива Cursor        |
| **Cline**          | VS Code ext       | ✅                        | Open-source, гибкий        |
| **Continue**       | VS Code/JetBrains | ✅                        | Мультиплатформенный        |
| **Open WebUI**     | Web               | ✅ Native                 | Pipes + Tools              |
| **LM Studio**      | Desktop           | ✅ Native                 | Встроенный MCP             |
| **Cherry Studio**  | Desktop           | ✅                        | Популярен в Азии           |
| **Jan**            | Desktop           | ✅                        | Локальный-first            |

---

## Настройка на NixOS

### Вариант 1: Через home-manager (рекомендуется)

```nix
# home.nix
{ pkgs, ... }:
{
  home.packages = with pkgs; [
    nodejs_22        # Для npx-based серверов
    uv               # Для Python-серверов
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

### Вариант 3: Flake с собственными MCP-пакетами

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

## Интеграция с Ollama

MCP - протокол транспорта. Ollama - движок LLM. Совместно дают локальный AI-стек без интернета и API-ключей.

### Шаг 1. Запустить Ollama

```bash
ollama pull qwen2.5-coder:7b  # Оптимальный выбор для tool calling на 16 GB RAM
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

**В Open WebUI** (через pipes):
Используются manifold-пайпы (Naga, Pollinations, OpenRouter, Cloudflare) + параметр `__tools__`.

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

## Практические сценарии

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

### Сценарий 4: Персональный секретарь

**Стек:** `notion` + `todoist` + `slack` + `memory`

- Синхронизация задач
- Саммаризация Slack-тредов
- Планирование дня

---

**Оптимизации для 16 GB RAM:**

1. Использовать квантизации **Q4_K_M** (не fp16)
2. Модели: `qwen2.5-coder:7b`, `llama3.1:8b`, `mistral:7b`
3. Не держать все MCP-серверы постоянно запущенными - запускать по необходимости
4. Ограничить `num_ctx` в Ollama: `OLLAMA_NUM_CTX=8192`
5. Закрывать GUI-клиенты в простое (Cursor занимает 2+ GB)

---

## Ссылки

- [Официальная спецификация MCP](https://modelcontextprotocol.io)
- [Anthropic MCP Servers](https://github.com/modelcontextprotocol/servers)
- [Awesome MCP Servers](https://github.com/punkpeye/awesome-mcp-servers)
- [Ollama + MCP](https://github.com/ollama/ollama)
- [Cursor MCP Docs](https://docs.cursor.com/context/model-context-protocol)
- [MCP на NixOS (community)](https://github.com/nix-community/mcp-servers)
