---
author: ["Potato Energy Team", "ponfertato"]
categories: ["docker", "backup", "guide"]
date: "2026-03-16T16:15:00+03:00"
description: "Бэкап, восстановление и миграция Docker Volumes. Практическое руководство с примерами."
draft: false
series: ["Docker"]
slug: "volumes-management"
tags: ["docker", "volumes", "backup", "restore", "migration"]
title: "Docker Volumes: Бэкап и миграция"
---

Docker Volumes хранят данные независимо от контейнеров, но требуют отдельного подхода к резервному копированию. Это руководство описывает универсальные методы работы с volumes на примере популярных сервисов.

> 💡 Заменяйте имена volumes на свои. Методы работают с любыми контейнерами.

---

## 📦 Бэкап volumes

### Через docker-volume-backup (рекомендуется)

```bash
docker run --rm \
  -v portainer_data:/backup/portainer_data \
  -v postgres_data:/backup/postgres_data \
  -v redis_data:/backup/redis_data \
  -v /opt/docker/backup:/archive \
  --entrypoint backup \
  offen/docker-volume-backup:v2
```

**Параметры**:
| Параметр | Описание |
|----------|----------|
| `-v <volume>:/backup/<name>` | Маппинг volume в директорию бэкапа |
| `-v /opt/docker/backup:/archive` | Куда сохранять архив на хосте |
| `--entrypoint backup` | Запускает режим бэкапа |
| `--rm` | Удаляет контейнер после завершения |

**Почему этот метод**:

- ✅ Один контейнер - все volumes
- ✅ Автоматическое сжатие в tar.gz
- ✅ Не требует остановки сервисов (для большинства сценариев)
- ✅ Подходит для автоматизации по cron

### Альтернатива: через tar напрямую

```bash
# Бэкап одного volume
docker run --rm \
  -v postgres_data:/volume:ro \
  -v /opt/docker/backup:/backup \
  alpine tar czf /backup/postgres-$(date +%F).tar.gz -C /volume .
```

**Особенности**:

- `:ro` - read-only, защита от модификации во время бэкапа
- Прямое создание архива без промежуточных скриптов
- Подходит для разовых операций

---

## 🔄 Восстановление из бэкапа

### Шаг 1: Распаковать архив

```bash
tar -C /tmp -xvf backup-*.tar.gz
```

### Шаг 2: Копирование данных в volumes

```bash
# Portainer
docker run -d --name tmp-portainer -v portainer_data:/restore alpine
docker cp /tmp/backup/portainer_data tmp-portainer:/restore
docker stop tmp-portainer && docker rm tmp-portainer

# PostgreSQL
docker run -d --name tmp-postgres -v postgres_data:/restore alpine
docker cp /tmp/backup/postgres_data tmp-postgres:/restore
docker stop tmp-postgres && docker rm tmp-postgres

# Redis
docker run -d --name tmp-redis -v redis_data:/restore alpine
docker cp /tmp/backup/redis_data tmp-redis:/restore
docker stop tmp-redis && docker rm tmp-redis
```

**Почему через временные контейнеры**:

- Прямой доступ к volumes из хоста сложен (путь в `/var/lib/docker/volumes/`)
- `docker cp` сохраняет права и атрибуты файлов
- Контейнеры удаляются после копирования - чисто и безопасно

### ⚠️ Рекомендация

```bash
# Остановить сервисы перед восстановлением
docker compose stop postgres redis portainer

# Выполнить восстановление

# Запустить сервисы
docker compose start
```

---

## 🔀 Миграция volumes

### Копирование в новый volume

```bash
docker volume create new_volume && \
docker run --rm -it \
  -v old_volume:/from \
  -v new_volume:/to \
  alpine ash -c 'cd /from && cp -av . /to'
```

**Разбор**:

- `docker volume create` - создаёт целевой volume
- `-v old_volume:/from` - монтирует источник
- `-v new_volume:/to` - монтирует приёмник
- `cp -av` - копирует с сохранением прав (`-a`) и прогрессом (`-v`)

**Когда использовать**:

- Переименование volume
- Перенос на другой драйвер хранения
- Очистка от временных файлов (копируем только нужное)

### Миграция на другой хост

```bash
# Исходный хост: создать архив
docker run --rm \
  -v postgres_data:/data \
  -v /backup:/archive \
  alpine tar czf /archive/postgres.tar.gz -C /data .

# Скопировать на новый хост
scp /backup/postgres.tar.gz user@newhost:/backup/

# Новый хост: восстановить
docker volume create postgres_data
docker run --rm \
  -v postgres_data:/data \
  -v /backup:/archive \
  alpine tar xzf /archive/postgres.tar.gz -C /data
```

**Преимущества**:

- Переносит все данные и права
- Не зависит от версий Docker
- Работает через любую сеть (SCP, rsync, HTTPS)

---

## 🛠 Автоматизация

### Через cron на хосте

```bash
# /etc/crontabs/root - ежедневный бэкап в 03:00
0 3 * * * docker run --rm -v postgres_data:/backup/data -v /opt/backup:/archive --entrypoint backup offen/docker-volume-backup:v2
```

### Через Docker Compose

```yaml
version: "3.8"
services:
  backup:
    image: offen/docker-volume-backup:v2
    container_name: volume-backup
    environment:
      BACKUP_CRON_EXPRESSION: "0 3 * * *"
      BACKUP_RETENTION_DAYS: "7"
    volumes:
      - postgres_data:/backup/postgres_data
      - redis_data:/backup/redis_data
      - /opt/backup:/archive
    entrypoint: ["/bin/sh", "-c", "backup"]
```

**Возможности**:

- Встроенная ротация бэкапов
- Логирование внутри контейнера
- Поддержка S3-совместимых хранилищ

---

## 📊 Проверка целостности

```bash
# После бэкапа: проверить архив
tar -tzf backup-*.tar.gz | head -20
sha256sum backup-*.tar.gz > backup.sha256

# После восстановления: проверить данные
docker system df -v | grep postgres_data
docker logs postgres --tail 50
```

---

## ⚠️ Частые проблемы

```bash
# "volume not found"
→ docker volume ls | grep <name>
→ docker volume inspect <name>

# "permission denied"
→ Запускать от root или с sudo
→ Проверить права: ls -la /opt/docker/backup/

# Бэкап слишком большой
→ Исключить кэш: tar --exclude='cache/*' ...
→ Настроить ротацию: BACKUP_RETENTION_DAYS=7

# Восстановление не работает
→ Остановить сервис: docker stop <container>
→ Проверить, что volume не используется: docker ps --filter volume=<name>
```

---

## 🗂 Чеклист

### Перед бэкапом

- [ ] Проверил место: `df -h /opt/docker/backup`
- [ ] Остановил БД и критичные сервисы
- [ ] Протестировал на одном volume

### Перед восстановлением

- [ ] Остановил целевые контейнеры
- [ ] Проверил архив: `tar -tzf backup.tar.gz`
- [ ] Подготовил план отката

---

## Ссылки

- 🐳 [Docker Volumes Docs](https://docs.docker.com/storage/volumes/)
- 🔄 [offen/docker-volume-backup](https://github.com/offen/docker-volume-backup)
- 📦 [Docker Compose Production](https://docs.docker.com/compose/production/)
