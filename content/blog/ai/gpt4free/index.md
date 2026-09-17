---
author: ["Potato Energy Team", "ponfertato"]
categories: ["gpt", "ai", "tutorial"]
date: "2026-03-16T15:25:00+03:00"
description: "Практическое руководство по GPT4Free: установка, настройка и использование бесплатных аналогов GPT-4/5 через Python и TypeScript."
draft: false
series: ["GPT4Free"]
slug: "gpt4free"
tags: ["git", "gpt", "python", "typescript", "api", "docker"]
title: "GPT4Free: доступ к GPT-5, DeepSeek и Gemini"
aliases:
  - /blog/gpt/gpt4free/
---

GPT4Free (g4f) - инструмент, предоставляющий доступ к моделям GPT-4/5, Claude, Gemini, DeepSeek через обратную инженерию публичных API.

> ⚠️ **Важно**: только для обучения и тестов. Может нарушать правила некоторых сервисов.

---

## Установка

### Требования

- Компьютер с интернетом
- [Python 3.10+](https://python.org) (при установке отметить "Add to PATH")

### Команда установки

```bash
pip install -U g4f[all]
```

---

## Запуск

### Вариант 1: Веб-интерфейс

```bash
python -m g4f.cli gui --port 8080
```

Интерфейс: `http://localhost:8080/chat/`

### Вариант 2: Локальный API

```bash
python -m g4f --port 1337
```

После запуска к API можно подключать приложения, поддерживающие OpenAI API.

---

## Первый скрипт

Создать файл `test.py`:

```python
from g4f.client import Client

client = Client()

response = client.chat.completions.create(
    model="gpt-4o-mini",
    messages=[{"role": "user", "content": "Почему картофель - это состояние души?"}]
)

print(response.choices[0].message.content)
```

Запуск:

```bash
python test.py
```

---

## Генерация изображений

```python
from g4f.client import Client

client = Client()

img = client.images.generate(
    model="flux",
    prompt="Киберпанк-картофель в неоновом городе",
    response_format="url"
)

print(f"Готово: {img.data[0].url}")
```

---

## Актуальные модели (март 2026)

| Модель           | Статус         | Назначение              |
| ---------------- | -------------- | ----------------------- |
| `gpt-4o-mini`    | ✅ Стабильно   | Быстрые ответы, чат     |
| `gpt-4o`         | ✅ Стабильно   | Сложные задачи          |
| `deepseek-v3`    | ✅ Стабильно   | Код, логика, математика |
| `gemini-2.5-pro` | ⚠️ Нестабильно | Мультимодальные задачи  |
| `llama-3.3-70b`  | ✅ Стабильно   | Открытая альтернатива   |
| `gpt-5`          | 🔶 Эксперимент | Может не работать       |

> 💡 Список меняется. Актуальный список - через веб-интерфейс или запрос:
> `GET http://localhost:8080/backend-api/models`

---

## Подключение к OpenAI-совместимым приложениям

После запуска `python -m g4f --port 1337`:

```python
from openai import OpenAI

client = OpenAI(
    base_url="http://localhost:1337/v1",
    api_key="не-важно-что-здесь"  # можно любое значение
)

response = client.chat.completions.create(
    model="gpt-4o-mini",
    messages=[{"role": "user", "content": "Расскажи анекдот про картофель"}]
)

print(response.choices[0].message.content)
```

Совместимо с LibreChat, Flowise, AnythingLLM и другими.

---

## Диагностика

```bash
# Обновить библиотеку
pip install -U g4f

# Ошибка при установке на Windows
pip install --upgrade pip setuptools wheel

# Модель не отвечает
# → Использовать другую модель
# → Включить VPN
# → Подождать 10-30 секунд (некоторые провайдеры медленные)

# При использовании Docker добавить памяти браузеру:
docker run -p 8080:8080 --shm-size="2g" hlohaus789/g4f:latest
```

---

## Docker

```bash
docker run -p 8080:8080 --shm-size="2g" hlohaus789/g4f:latest
```

- Веб-интерфейс: `http://localhost:8080`
- API: `http://localhost:8080/v1`

---

## Ссылки

- [Официальный репозиторий](https://github.com/xtekky/gpt4free)
- [Docker-образ](https://hub.docker.com/r/hlohaus789/g4f)
- [Документация](https://g4f.dev)
- [Telegram-канал](https://t.me/g4f_channel)
