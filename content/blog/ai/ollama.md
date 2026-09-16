---
author: ["Potato Energy Team", "ponfertato"]
categories: ["ai", "ollama", "guide"]
date: "2026-09-16T19:00:00+03:00"
description: "Ollama: установка, настройка, интеграции с Open WebUI и MCP. Выбор моделей под разное железо, советы по производительности."
draft: false
series: ["Tools"]
slug: "ollama"
tags: ["ollama", "llm", "ai", "open-webui", "mcp", "local-ai"]
title: "Ollama: Локальные нейросети без боли"
---

Ollama - это рантайм для запуска больших языковых моделей локально. Одна команда - и модель работает. Никакой возни с зависимостями, виртуальными окружениями и конфигурацией.

```
Твоя машина → Ollama → Модель (Llama, Qwen, Mistral...)
```

**Почему это важно:**

- Данные не покидают машину (приватность)
- Не нужен интернет для инференса
- Нет подписок и лимитов
- Работает на CPU (медленно, но работает)

---

## Установка

### NixOS

```nix
# configuration.nix
services.ollama = {
enable = true;
# Опционально: укажи модели для автозагрузки
# models = [ "qwen2.5:7b" ];
};
```

Применение:

```bash
sudo nixos-rebuild switch
```

### Linux (Debian/Ubuntu)

```bash
curl -fsSL https://ollama.com/install.sh | sh
```

### Windows

1. Скачай установщик с [ollama.com](https://ollama.com/download)
2. Установи как обычное приложение
3. Проверь в терминале: `ollama --version`

### Проверка установки

```bash
ollama --version
ollama serve   # Запуск сервера (если не автозапуск)
```

---

## Базовое использование

### Запуск модели

```bash
# Интерактивный чат
ollama run qwen2.5:7b

# Один запрос
ollama run qwen2.5:7b "Напиши функцию на Python для сортировки"

# С системным промптом
ollama run qwen2.5:7b --system "Ты эксперт по Linux" "Как настроить firewall?"
```

### Управление моделями

```bash
# Список установленных моделей
ollama list

# Скачать модель без запуска
ollama pull qwen2.5:7b

# Удалить модель
ollama rm qwen2.5:7b

# Информация о модели
ollama show qwen2.5:7b
```

### Доступные модели

| Модель           | Размер  | Для чего                    |
| ---------------- | ------- | --------------------------- |
| `qwen2.5:7b`     | ~4.5 GB | Универсальная, код, русский |
| `llama3.2:3b`    | ~2 GB   | Быстрая, простые задачи     |
| `mistral:7b`     | ~4 GB   | Универсальная, быстрая      |
| `codellama:7b`   | ~4 GB   | Специализация на коде       |
| `gemma2:9b`      | ~5.5 GB | Качественные ответы         |
| `llama3.1:8b`    | ~4.7 GB | Универсальная               |
| `deepseek-r1:8b` | ~5 GB   | Рассуждения, логика         |

---

## Настройка

### Параметры запуска

```bash
# Увеличить контекст
ollama run qwen2.5:7b --num-ctx 8192

# Температура (креативность)
ollama run qwen2.5:7b --temperature 0.7

# Максимум токенов в ответе
ollama run qwen2.5:7b --num-predict 1024
```

### Переменные окружения

```bash
# Указать порт (по умолчанию 11434)
OLLAMA_HOST=0.0.0.0:11434 ollama serve

# Указать директорию для моделей
OLLAMA_MODELS=/path/to/models ollama serve

# Количество потоков CPU
OLLAMA_NUM_PARALLEL=4 ollama serve
```

В NixOS:

```nix
services.ollama = {
enable = true;
host = "0.0.0.0:11434";  # Доступ из сети
};
```

### Кастомная модель (Modelfile)

Создай файл `Modelfile`:

```
FROM qwen2.5:7b
SYSTEM Ты эксперт по Linux и DevOps. Отвечай кратко и по делу.
PARAMETER temperature 0.3
PARAMETER num_ctx 8192
```

Создай модель:

```bash
ollama create my-expert -f Modelfile
ollama run my-expert "Как настроить docker-compose?"
```

---

## API

### Локальный API

Ollama поднимает HTTP API на порту 11434:

```bash
# Список моделей
curl http://localhost:11434/api/tags

# Генерация
curl http://localhost:11434/api/generate -d '{
"model": "qwen2.5:7b",
"prompt": "Привет"
}'

# Чат (как в OpenAI)
curl http://localhost:11434/api/chat -d '{
"model": "qwen2.5:7b",
"messages": [{"role": "user", "content": "Привет"}]
}'
```

### OpenAI-совместимый API

Ollama эмулирует OpenAI API:

```bash
curl http://localhost:11434/v1/chat/completions \
-H "Content-Type: application/json" \
-d '{
"model": "qwen2.5:7b",
"messages": [{"role": "user", "content": "Привет"}]
}'
```

Использование в Python:

```python
from openai import OpenAI

client = OpenAI(
base_url="http://localhost:11434/v1",
api_key="ollama"  # любой текст
)

response = client.chat.completions.create(
model="qwen2.5:7b",
messages=[{"role": "user", "content": "Привет"}]
)
print(response.choices[0].message.content)
```

---

## Интеграции

### Open WebUI

Open WebUI нативно поддерживает Ollama.

1. Запусти Ollama: `ollama serve`
2. Открой Open WebUI
3. Настройки → Подключения → Добавь:

```
URL: http://localhost:11434
```

Если в Docker:

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

### MCP-серверы

Ollama можно использовать как LLM для MCP-клиентов. Пример для MCP-клиента:

```json
{
  "model": "qwen2.5:7b",
  "api_base": "http://localhost:11434/v1",
  "api_key": "ollama"
}
```

### Continue.dev

В конфиге `~/.continue/config.json`:

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

### Другое

| Инструмент      | Как подключить                                                                                |
| --------------- | --------------------------------------------------------------------------------------------- |
| **LibreChat**   | В конфиге `librechat.yaml` добавить `endpoints.custom` с `baseURL: http://localhost:11434/v1` |
| **Flowise**     | В настройках ноды указать `baseURL: http://localhost:11434`                                   |
| **AnythingLLM** | В настройках подключения выбрать Ollama                                                       |

---

## Выбор модели под железо

### Без GPU (CPU only)

| RAM   | Модели                                     | Скорость     |
| ----- | ------------------------------------------ | ------------ |
| 8 GB  | `llama3.2:3b`, `qwen2.5:3b`                | ~10-15 ток/с |
| 16 GB | `qwen2.5:7b`, `mistral:7b`, `llama3.1:8b`  | ~5-10 ток/с  |
| 32 GB | `qwen2.5:14b`, `llama3.1:8b` (квант. выше) | ~3-5 ток/с   |
| 64 GB | `qwen2.5:32b`, `llama3.1:70b` (Q4)         | ~1-3 ток/с   |

### С GPU

| VRAM  | Модели                | Скорость     |
| ----- | --------------------- | ------------ |
| 8 GB  | `qwen2.5:7b` (полный) | ~30-50 ток/с |
| 12 GB | `qwen2.5:14b`         | ~20-30 ток/с |
| 24 GB | `qwen2.5:32b`         | ~10-15 ток/с |

**Совет для 16 GB без GPU:**

- Используй `qwen2.5:7b` с квантизацией по умолчанию (Q4)
- Контекст 4096-8192 токенов
- Температура 0.3-0.7

---

## Советы по производительности

### 1. Квантизация

Чем ниже квантизация, тем меньше памяти, но хуже качество:

```bash
# Пример: qwen2.5:7b-instruct-q4_K_M
ollama pull qwen2.5:7b-instruct-q4_K_M
```

| Квант.   | Размер  | Качество             |
| -------- | ------- | -------------------- |
| `q8_0`   | ~8 GB   | Отличное             |
| `q5_K_M` | ~5.5 GB | Хорошее              |
| `q4_K_M` | ~4.5 GB | Хорошее (рекомендую) |
| `q3_K_M` | ~3.5 GB | Среднее              |
| `q2_K`   | ~2.5 GB | Низкое               |

### 2. Контекст

```bash
# Увеличить контекст (требует больше памяти)
ollama run qwen2.5:7b --num-ctx 8192
```

### 3. Количество потоков

```bash
# Указать количество потоков (по умолчанию = все ядра)
OLLAMA_NUM_PARALLEL=4 ollama serve
```

---

## Частые проблемы

| Проблема                    | Решение                                            |
| --------------------------- | -------------------------------------------------- |
| `ollama: command not found` | Перезапусти терминал или добавь в PATH             |
| Модель не скачивается       | Проверь интернет, место на диске                   |
| Медленная генерация на CPU  | Уменьши модель (3b вместо 7b), уменьши контекст    |
| `connection refused`        | Проверь, запущен ли `ollama serve`                 |
| Open WebUI не видит модели  | Проверь `OLLAMA_BASE_URL`, перезапусти оба сервиса |
| Модель "забывает" контекст  | Увеличь `--num-ctx`                                |
| Out of memory               | Уменьши модель или квантизацию                     |

---

## Ссылки

- 🦙 [Ollama официальный сайт](https://ollama.com)
- 📚 [Документация](https://github.com/ollama/ollama)
- 🗂️ [Библиотека моделей](https://ollama.com/library)
- 🌐 [Open WebUI](https://github.com/open-webui/open-webui)
- 🔗 [MCP Protocol](https://modelcontextprotocol.io)
