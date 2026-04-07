---
author: ["Potato Energy Team", "ponfertato"]
categories: ["git", "github", "automation", "guide"]
date: "2026-03-17T09:30:00+03:00"
description: "Массовое удаление старых GitHub Issues по меткам и дате через GitHub CLI и jq."
draft: false
series: ["Git Tips"]
slug: "delete-issues"
tags: ["git", "github", "cli", "jq", "automation", "cleanup"]
title: "GitHub CLI: Массовое удаление старых задач"
---

В долгосрочных проектах накапливаются устаревшие задачи: баги для старых версий, фичи, которые уже не актуальны, тестовые тикеты. Ручное удаление - долго и скучно. Этот скрипт автоматически находит и удаляет (или помечает) старые задачи по заданным меткам.

> 💡 Скрипт использует **dry-run по умолчанию** - сначала покажет, что будет удалено, без реального удаления.

---

## 📦 Скрипт: delete-issues.sh

### Полный код

```bash
#!/bin/bash
# Удаление старых GitHub Issues по меткам и дате
# Использование: ./delete-issues.sh [--execute]

set -euo pipefail

# === НАСТРОЙКИ ===
REPO="owner/repo"                          # Репозиторий в формате owner/repo
LABELS='label1,label2,label3'              # Метки для фильтрации (через запятую)
CUTOFF="2025-12-31T23:59:59Z"             # Удалять задачи, созданные ДО этой даты
DRY_RUN=true                               # true = только показать, false = реально удалить

# Парсинг аргументов
if [[ "${1:-}" == "--execute" ]]; then
    DRY_RUN=false
    echo "⚠️  Режим: РЕАЛЬНОЕ УДАЛЕНИЕ"
else
    echo "ℹ️  Режим: DRY RUN (ничего не будет удалено)"
fi

echo "🔍 Поиск задач в $REPO с метками: $LABELS, созданных до $CUTOFF"
echo "---"

# Получение и фильтрация задач
gh issue list --repo "$REPO" --state all --limit 1000 \
  --json number,title,createdAt,labels,url | \
  jq -r --arg labels "$LABELS" --arg cutoff "$CUTOFF" '
    ($labels | split(",")) as $label_array |
    .[] |
    select(.createdAt < $cutoff) |
    select(.labels | map(.name) | any(. as $l | $label_array | index($l))) |
    "\(.number)|\(.title)|\(.url)"
  ' | \
  while IFS='|' read -r number title url; do
    if [[ "$DRY_RUN" == "true" ]]; then
      echo "[DRY RUN] ##$number - $title"
      echo "          $url"
    else
      echo "🗑  Удаление ##$number - $title"
      gh issue delete "$REPO" "$number" --yes
      sleep 1  # Небольшая пауза, чтобы не превысить rate limit
    fi
  done

echo "---"
echo "✅ Готово"
```

### Установка зависимостей

```bash
# GitHub CLI
# Windows (winget):
winget install GitHub.cli

# Linux (Ubuntu/Debian):
sudo apt install gh

# macOS (Homebrew):
brew install gh

# Авторизация
gh auth login

# jq (JSON-процессор)
# Windows (winget):
winget install jq.jq

# Linux:
sudo apt install jq

# macOS:
brew install jq
```

### Запуск

```bash
# Сделать скрипт исполняемым
chmod +x delete-issues.sh

# DRY RUN (безопасный режим - только показать)
./delete-issues.sh

# РЕАЛЬНОЕ УДАЛЕНИЕ (добавить флаг --execute)
./delete-issues.sh --execute
```

---

## 🔍 Как это работает

### Пошаговый разбор

| Шаг | Команда                    | Что делает                                                                     |
| --- | -------------------------- | ------------------------------------------------------------------------------ |
| 1   | `gh issue list --json ...` | Получает задачи в формате JSON с полями: номер, заголовок, дата, метки, ссылка |
| 2   | `jq -r ...`                | Фильтрует: дата < cutoff И есть хотя бы одна из указанных меток                |
| 3   | `while read ...`           | Обрабатывает каждую найденную задачу                                           |
| 4   | `gh issue delete`          | Удаляет задачу (только если `DRY_RUN=false`)                                   |

### Логика jq-фильтра

```jq
($labels | split(",")) as $label_array |  # Разбиваем строку меток в массив
.[] |                                      # Для каждой задачи
select(.createdAt < $cutoff) |            # Только если создана до cutoff
select(
  .labels | map(.name) |                  # Получаем список имён меток
  any(. as $l | $label_array | index($l)) # Есть ли совпадение с нашими метками
) |
"\(.number)|\(.title)|\(.url)"            # Форматируем вывод
```

**Почему так**:

- `split(",")` - позволяет задавать метки одной строкой
- `any(...)` - проверяет наличие **любой** из указанных меток (логическое ИЛИ)
- Формат вывода через `|` - надёжный способ передать заголовок с пробелами

---

## ⚙️ Адаптация под свои задачи

### Пример 1: Удалить старые баги

```bash
REPO="myorg/myproject"
LABELS='bug,critical'
CUTOFF="2024-01-01T00:00:00Z"  # Удалить баги до 2024 года
```

### Пример 2: Архивировать, а не удалять

Замените блок удаления на закрытие:

```bash
# Вместо gh issue delete:
echo "🔒 Закрытие ##$number - $title"
gh issue edit "$REPO" "$number" --state closed
```

### Пример 3: Экспорт перед удалением (резервная копия)

```bash
# Добавить перед удалением:
echo "💾 Экспорт ##$number"
gh issue view "$REPO" "$number" --json title,body,comments >> backup-issues.jsonl
```

---

## 🛡 Безопасность

### Обязательные проверки перед запуском

```bash
# 1. Проверить, что выбранные задачи - действительно те, что нужно
./delete-issues.sh | grep -c "\[DRY RUN\]"   # Посчитать количество

# 2. Убедиться, что в списке нет важных задач
./delete-issues.sh | grep -i "critical\|important"

# 3. Проверить права доступа
gh api /repos/$REPO | jq '.permissions.admin'  # Должно быть true
```

### Rate limits GitHub API

| Тип             | Лимит             | Как не превысить                    |
| --------------- | ----------------- | ----------------------------------- |
| Авторизованный  | 5000 запросов/час | `sleep 1` между удалениями          |
| Без авторизации | 60 запросов/час   | Всегда использовать `gh auth login` |

**Проверить остаток лимита**:

```bash
gh api rate_limit | jq '.resources.core'
```

---

## 📊 Альтернативы и расширения

### Только показать статистику (без удаления)

```bash
gh issue list --repo "$REPO" --state all --limit 1000 \
  --json createdAt,labels | \
  jq -r --arg labels "$LABELS" --arg cutoff "$CUTOFF" '
    ($labels|split(",")) as $label_array |
    [.[] | select(.createdAt < $cutoff) |
     select(.labels | map(.name) | any(. as $l | $label_array | index($l)))] |
    length
  '
```

### Удаление по автору + меткам

```jq
select(.author.login == "bot-user") |
select(.labels | map(.name) | any(. as $l | $label_array | index($l)))
```

### Экспорт в CSV перед удалением

```bash
# Добавить в jq-вывод:
"\(.number),\"\(.title)\",\(.createdAt),\(.url)"
```

---

## ⚠️ Частые проблемы

```bash
# "gh: command not found"
→ Установить GitHub CLI: https://cli.github.com

# "jq: error: syntax error"
→ Проверить кавычки в команде: использовать одинарные для jq, двойные для bash

# "HTTP 404: Not Found"
→ Проверить имя репозитория: должно быть owner/repo
→ Убедиться, что у токена есть права на чтение issues

# "rate limit exceeded"
→ Подождать сброса лимита или добавить задержку: sleep 2

# Удалил не то!
→ Всегда запускать сначала без --execute
→ Делать бэкап: gh issue list --json ... > backup.json
```

---

## 🗂 Чеклист перед запуском

- [ ] Установил `gh` и `jq`, выполнил `gh auth login`
- [ ] Проверил права на репозиторий: `gh api /repos/owner/repo`
- [ ] Протестировал в dry-run режиме, проверил список задач
- [ ] Убедился, что в списке нет критичных задач
- [ ] При необходимости - сделал экспорт: `gh issue list --json ... > backup.json`
- [ ] Только после этого - запуск с `--execute`

---

## Ссылки

- 🐙 [GitHub CLI Docs](https://cli.github.com/manual/)
- 🔍 [gh issue list](https://cli.github.com/manual/gh_issue_list)
- 🗑️ [gh issue delete](https://cli.github.com/manual/gh_issue_delete)
- 🧰 [jq Manual](https://jqlang.github.io/jq/manual/)
- 🔐 [GitHub API Rate Limits](https://docs.github.com/en/rest/rate-limit)
