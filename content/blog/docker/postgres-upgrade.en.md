---
author: ["Potato Energy Team", "ponfertato"]
categories: ["docker", "postgres", "migration", "guide"]
date: "2026-04-22T17:30:00+03:00"
description: "Safe PostgreSQL major version migration in Docker: from 15 to 17/18. Accounts for v18+ breaking change, step-by-step dump/restore."
draft: false
series: ["Docker"]
slug: "postgres-upgrade"
tags: ["docker", "postgres", "pgdata", "migration", "backup", "pg_dumpall"]
title: "PostgreSQL in Docker: Major Version Migration Without Data Loss"
---

PostgreSQL doesn't support in-place upgrades between major versions (15 → 17, 17 → 18). Data must be migrated logically: via dump and restore.

**Why it's important**:

- ✅ New versions = security fixes, optimizations, new features
- ✅ Support for modern clients (Nextcloud, Mastodon, Authelia require recent versions)
- ✅ Predictability: same process for any project

**Docker complexity**:

- ❌ Can't just change image tag - data format is incompatible
- ❌ `pg_upgrade` requires simultaneous access to old/new binaries - breaks container isolation
- ✅ Solution: `pg_dumpall` → new container → `psql < dump`

---

## 📋 Prerequisites

Before starting, ensure:

- [ ] Terminal access with `docker` and `docker compose` permissions
- [ ] ~2× database size free on disk (for dump + archive)
- [ ] Know DB container name (`nextcloud-postgres`) and user (`nextcloud`)
- [ ] Have `docker-compose.yml` with `postgres` service definition
- [ ] Application (Nextcloud/Mastodon) is stopped or in maintenance mode

> 💡 When in doubt - make a full volume backup first: `tar czf backup-volume.tar.gz /var/lib/docker/volumes/...`

---

### Step 1: Logical Backup

```bash
# Create dump of all databases, roles, and global settings
docker exec nextcloud-postgres pg_dumpall -U nextcloud > pg_dumpall_$(date +%F).sql

# Verify size and integrity
ls -lh pg_dumpall_*.sql
tail -10 pg_dumpall_*.sql  # should end with ";"
```

> ⚠️ If the user lacks superuser rights, use `-U postgres`.

### Step 2: Physical Volume Archive (Insurance)

```bash
# Stop DB
docker compose stop postgres

# Find volume name
docker volume ls | grep nextcloud_database

# Archive raw data
sudo tar czf postgres-volume-$(date +%F).tar.gz \
  -C /var/lib/docker/volumes/nextcloud_database/_data .
```

### Step 3: Remove Old Container and Volume

```bash
docker compose down postgres
docker volume rm nextcloud_database
```

### Step 4: Update `docker-compose.yml`

Choose **one** option based on your target version.

#### ✅ Option A: PostgreSQL 15 / 16 / 17 (no path changes)

```yaml
services:
  postgres:
    image: postgres:17-alpine # or 15, 16
    volumes:
      - database:/var/lib/postgresql/data
```

#### ✅ Option B: PostgreSQL 18+ (new standard)

```yaml
services:
  postgres:
    image: postgres:18-alpine
    volumes:
      - database:/var/lib/postgresql # ← mount parent directory
    # Container will auto-create /var/lib/postgresql/18/docker inside
```

#### 🔧 Option C: PostgreSQL 18+ with backward compatibility

```yaml
services:
  postgres:
    image: postgres:18-alpine
    volumes:
      - database:/var/lib/postgresql/data
    environment:
      PGDATA: /var/lib/postgresql/data # ← force old path
```

> 💡 **Recommendation**: use **Option B**. It aligns with Debian/Alpine standards and simplifies future `pg_upgrade --link`.

### Step 5: Start New DB

```bash
docker compose up -d postgres
sleep 20  # wait for initdb
docker exec nextcloud-postgres pg_isready -U nextcloud -d nextcloud
# Expected: "accepting connections"
```

### Step 6: Restore Data

```bash
docker exec -i nextcloud-postgres psql -U nextcloud < pg_dumpall_*.sql
```

> 🟡 **Normal errors in logs**:
>
> ```
> ERROR: role "nextcloud" already exists
> ERROR: database "nextcloud" already exists
> ```
>
> The new container already created the role/DB from `stack.env`. `psql` ignores duplicates and correctly imports tables. Just continue.

### Step 7: Final Verification

```bash
# Version
docker exec nextcloud-postgres psql -U nextcloud -c "SELECT version();"

# Data path
docker exec nextcloud-postgres psql -U nextcloud -c "SHOW data_directory;"
# ≤17: /var/lib/postgresql/data
# ≥18: /var/lib/postgresql/18/docker

# Tables
docker exec nextcloud-postgres psql -U nextcloud -d nextcloud -c "\dt"
```

---

## 🔄 Rollback (If Something Goes Wrong)

```bash
docker compose down

# Revert old image and path (example for v17)
sed -i 's/postgres:18/postgres:17/' docker-compose.yml
sed -i 's|/var/lib/postgresql$|/var/lib/postgresql/data|' docker-compose.yml

# Restore volume
sudo rm -rf /var/lib/docker/volumes/nextcloud_database/_data/*
sudo tar xzf postgres-volume-*.tar.gz -C /var/lib/docker/volumes/nextcloud_database/_data/

docker compose up -d postgres
```

---

## ⚠️ FAQ

```bash
# "What happens if I run 18 with volumes:/var/lib/postgresql/data?"
→ Container won't find data at the new path, runs initdb, and creates an empty DB.
  Old files remain untouched in the volume.

# "How to check the path after startup?"
→ SHOW data_directory; inside psql.

# "Can I use pg_upgrade instead of dump?"
→ In Docker, it requires simultaneous mounting of old/new binaries, breaking container isolation.
  Dump/restore is more reliable for containerized environments.

# "Will 18→19 work the same way?"
→ Yes. Each major upgrade changes the subdirectory (18/docker → 19/docker).
  One `/var/lib/postgresql` volume will hold all versions in parallel.
```

---

## Links

- 🐘 [PGDATA Change Discussion](https://github.com/docker-library/postgres/issues/1258)
- 🐳 [Official PostgreSQL Docker Image](https://hub.docker.com/_/postgres)
- 🗄️ [pg_dumpall Reference](https://www.postgresql.org/docs/current/app-pg-dumpall.html)
- 🔄 [Nextcloud PostgreSQL Guide](https://docs.nextcloud.com/server/latest/admin_manual/configuration_database/)
