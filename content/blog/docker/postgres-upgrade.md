---
author: ["Potato Energy Team", "ponfertato"]
categories: ["docker", "postgres", "migration", "guide"]
date: "2026-04-22T17:55:00+03:00"
description: "Безопасная миграция PostgreSQL между мажорными версиями в Docker: от 15 до 17/18. Учёт breaking change v18+, пошаговый дамп/рестор."
draft: false
series: ["Docker"]
slug: "postgres-upgrade"
tags: ["docker", "postgres", "pgdata", "migration", "backup", "pg_dumpall"]
title: "PostgreSQL в Docker: Миграция между версиями без потери данных"
---

PostgreSQL не поддерживает in-place upgrade между мажорными версиями (15 → 17, 17 → 18). Данные нужно переносить логически: через дамп и восстановление.

**Почему это важно**:

- ✅ Новые версии = исправления безопасности, оптимизации, новые функции
- ✅ Поддержка актуальных клиентов (Nextcloud, Mastodon, Authelia требуют свежие версии)
- ✅ Предсказуемость: один и тот же процесс для любого проекта

**Сложность в Docker**:

- ❌ Нельзя просто поменять тег образа - формат данных несовместим
- ❌ `pg_upgrade` требует одновременного доступа к старым и новым бинарникам - ломает изоляцию контейнеров
- ✅ Решение: `pg_dumpall` → новый контейнер → `psql < dump`

---

## 📋 Предварительные требования

Перед началом убедись, что:

- [ ] Есть доступ к терминалу с правами на `docker` и `docker compose`
- [ ] Свободно ~2× размера базы на диске (для дампа + архива)
- [ ] Знаешь имя контейнера БД (`nextcloud-postgres`) и пользователя (`nextcloud`)
- [ ] Есть файл `docker-compose.yml` с описанием сервиса `postgres`
- [ ] Приложение (Nextcloud/Mastodon) остановлено или переведено в режим обслуживания

> 💡 Если не уверен - сделай полный бэкап тома перед началом: `tar czf backup-volume.tar.gz /var/lib/docker/volumes/...`

---

### Шаг 1: Логический бэкап

```bash
# Создаём дамп всех баз, ролей и глобальных настроек
docker exec nextcloud-postgres pg_dumpall -U nextcloud > pg_dumpall_$(date +%F).sql

# Проверяем размер и целостность
ls -lh pg_dumpall_*.sql
tail -10 pg_dumpall_*.sql  # должен заканчиваться ";"
```

> ⚠️ Если пользователь не имеет прав суперпользователя, используй `-U postgres`.

### Шаг 2: Физический архив тома (страховка)

```bash
# Останавливаем БД
docker compose stop postgres

# Находим имя тома
docker volume ls | grep nextcloud_database

# Архивируем сырые данные
sudo tar czf postgres-volume-$(date +%F).tar.gz \
  -C /var/lib/docker/volumes/nextcloud_database/_data .
```

### Шаг 3: Удаление старого контейнера и тома

```bash
docker compose down postgres
docker volume rm nextcloud_database
```

### Шаг 4: Обновление `docker-compose.yml`

Выбери **один** из вариантов в зависимости от целевой версии.

#### ✅ Вариант А: PostgreSQL 15 / 16 / 17 (без изменений путей)

```yaml
services:
  postgres:
    image: postgres:17-alpine # или 15, 16
    volumes:
      - database:/var/lib/postgresql/data
```

#### ✅ Вариант Б: PostgreSQL 18+ (новый стандарт)

```yaml
services:
  postgres:
    image: postgres:18-alpine
    volumes:
      - database:/var/lib/postgresql # ← монтируем родительскую директорию
    # Контейнер сам создаст /var/lib/postgresql/18/docker внутри тома
```

#### 🔧 Вариант В: PostgreSQL 18+ с обратной совместимостью

```yaml
services:
  postgres:
    image: postgres:18-alpine
    volumes:
      - database:/var/lib/postgresql/data
    environment:
      PGDATA: /var/lib/postgresql/data # ← принудительно старый путь
```

> 💡 **Рекомендация**: используй **Вариант Б**. Он соответствует стандарту Debian/Alpine и упрощает будущие `pg_upgrade --link`.

### Шаг 5: Запуск новой БД

```bash
docker compose up -d postgres
sleep 20  # ждём initdb
docker exec nextcloud-postgres pg_isready -U nextcloud -d nextcloud
# Ожидаемо: "accepting connections"
```

### Шаг 6: Восстановление данных

```bash
docker exec -i nextcloud-postgres psql -U nextcloud < pg_dumpall_*.sql
```

> 🟡 **Нормальные ошибки в логах**:
>
> ```
> ERROR: role "nextcloud" already exists
> ERROR: database "nextcloud" already exists
> ```
>
> Новый контейнер уже создал роль/БД из `stack.env`. `psql` игнорирует дубли и корректно заливает таблицы. Просто продолжай.

### Шаг 7: Финальная проверка

```bash
# Версия
docker exec nextcloud-postgres psql -U nextcloud -c "SELECT version();"

# Путь данных
docker exec nextcloud-postgres psql -U nextcloud -c "SHOW data_directory;"
# ≤17: /var/lib/postgresql/data
# ≥18: /var/lib/postgresql/18/docker

# Таблицы
docker exec nextcloud-postgres psql -U nextcloud -d nextcloud -c "\dt"
```

---

## 🔄 Откат (если что-то пошло не так)

```bash
docker compose down

# Возвращаем старый образ и путь (пример для 17)
sed -i 's/postgres:18/postgres:17/' docker-compose.yml
sed -i 's|/var/lib/postgresql$|/var/lib/postgresql/data|' docker-compose.yml

# Восстанавливаем том
sudo rm -rf /var/lib/docker/volumes/nextcloud_database/_data/*
sudo tar xzf postgres-volume-*.tar.gz -C /var/lib/docker/volumes/nextcloud_database/_data/

docker compose up -d postgres
```

---

## ⚠️ FAQ

```bash
# "Что будет, если запустить 18 с volumes:/var/lib/postgresql/data?"
→ Контейнер не найдёт данные по новому пути, выполнит initdb и создаст пустую БД.
  Старые файлы останутся в томе нетронутыми.

# "Как проверить путь после запуска?"
→ SHOW data_directory; внутри psql.

# "А можно использовать pg_upgrade вместо дампа?"
→ В Docker это требует одновременного маунта старых/новых бинарников, что ломает изоляцию.
  Dump/restore надёжнее для контейнерных сред.

# "Обновится ли 18→19 так же?"
→ Да. При каждом мажорном апгрейде меняется поддиректория (18/docker → 19/docker).
  Один том `/var/lib/postgresql` будет содержать все версии параллельно.
```

---

## Ссылки

- 🐘 [PGDATA Change Discussion](https://github.com/docker-library/postgres/issues/1258)
- 🐳 [Official PostgreSQL Docker Image](https://hub.docker.com/_/postgres)
- 🗄️ [pg_dumpall Reference](https://www.postgresql.org/docs/current/app-pg-dumpall.html)
- 🔄 [Nextcloud PostgreSQL Guide](https://docs.nextcloud.com/server/latest/admin_manual/configuration_database/)
