---
author: ["Potato Energy Team", "ponfertato"]
categories: ["docker", "backup", "guide"]
date: "2026-03-16T16:15:00+03:00"
description: "Backup, restore, and migration of Docker Volumes. Practical guide with examples."
draft: false
series: ["Docker"]
slug: "volumes-management"
tags: ["docker", "volumes", "backup", "restore", "migration"]
title: "Docker Volumes: Backup & Migration"
---

Docker Volumes store data independently from containers but require separate approach for backup. This guide describes universal methods for working with volumes using popular services as examples.

> 💡 Replace volume names with your own. Methods work with any containers.

---

## 📦 Backup volumes

### Via docker-volume-backup (recommended)

```bash
docker run --rm \
  -v portainer_data:/backup/portainer_data \
  -v postgres_data:/backup/postgres_data \
  -v redis_data:/backup/redis_data \
  -v /opt/docker/backup:/archive \
  --entrypoint backup \
  offen/docker-volume-backup:v2
```

**Parameters**:
| Parameter | Description |
|-----------|-------------|
| `-v <volume>:/backup/<name>` | Map volume to backup directory |
| `-v /opt/docker/backup:/archive` | Where to save archive on host |
| `--entrypoint backup` | Run backup mode |
| `--rm` | Remove container after completion |

**Why this method**:

- ✅ One container — all volumes
- ✅ Automatic tar.gz compression
- ✅ No service stop required (for most scenarios)
- ✅ Suitable for cron automation

### Alternative: via tar directly

```bash
# Single volume backup
docker run --rm \
  -v postgres_data:/volume:ro \
  -v /opt/docker/backup:/backup \
  alpine tar czf /backup/postgres-$(date +%F).tar.gz -C /volume .
```

**Features**:

- `:ro` — read-only, protects from modification during backup
- Direct archive creation without intermediate scripts
- Suitable for one-time operations

---

## 🔄 Restore from backup

### Step 1: Extract archive

```bash
tar -C /tmp -xvf backup-*.tar.gz
```

### Step 2: Copy data to volumes

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

**Why via temporary containers**:

- Direct volume access from host is complex (path in `/var/lib/docker/volumes/`)
- `docker cp` preserves file permissions and attributes
- Containers removed after copy — clean and safe

### Recommendation

```bash
# Stop services before restore
docker compose stop postgres redis portainer

# Perform restore

# Start services
docker compose start
```

---

## 🔀 Migrate volumes

### Copy to new volume

```bash
docker volume create new_volume && \
docker run --rm -it \
  -v old_volume:/from \
  -v new_volume:/to \
  alpine ash -c 'cd /from && cp -av . /to'
```

**Breakdown**:

- `docker volume create` — creates target volume
- `-v old_volume:/from` — mounts source
- `-v new_volume:/to` — mounts destination
- `cp -av` — copies with preserved permissions (`-a`) and progress (`-v`)

**When to use**:

- Renaming volume
- Migrating to different storage driver
- Cleaning from temp files (copy only needed data)

### Cross-host migration

```bash
# Source host: create archive
docker run --rm \
  -v postgres_data:/data \
  -v /backup:/archive \
  alpine tar czf /archive/postgres.tar.gz -C /data .

# Copy to new host
scp /backup/postgres.tar.gz user@newhost:/backup/

# New host: restore
docker volume create postgres_data
docker run --rm \
  -v postgres_data:/data \
  -v /backup:/archive \
  alpine tar xzf /archive/postgres.tar.gz -C /data
```

**Advantages**:

- Transfers all data and permissions
- Independent of Docker versions
- Works over any network (SCP, rsync, HTTPS)

---

## 🛠 Automation

### Via cron on host

```bash
# /etc/crontabs/root — daily backup at 03:00
0 3 * * * docker run --rm -v postgres_data:/backup/data -v /opt/backup:/archive --entrypoint backup offen/docker-volume-backup:v2
```

### Via Docker Compose

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

**Features**:

- Built-in backup rotation
- Logging inside container
- S3-compatible storage support

---

## 📊 Integrity verification

```bash
# After backup: verify archive
tar -tzf backup-*.tar.gz | head -20
sha256sum backup-*.tar.gz > backup.sha256

# After restore: verify data
docker system df -v | grep postgres_data
docker logs postgres --tail 50
```

---

## ⚠️ Common issues

```bash
# "volume not found"
→ docker volume ls | grep <name>
→ docker volume inspect <name>

# "permission denied"
→ Run as root or with sudo
→ Check permissions: ls -la /opt/docker/backup/

# Backup too large
→ Exclude cache: tar --exclude='cache/*' ...
→ Configure rotation: BACKUP_RETENTION_DAYS=7

# Restore not working
→ Stop service: docker stop <container>
→ Check volume not in use: docker ps --filter volume=<name>
```

---

## 🗂 Checklist

### Before backup

- [ ] Checked space: `df -h /opt/docker/backup`
- [ ] Stopped DB and critical services
- [ ] Tested on one volume

### Before restore

- [ ] Stopped target containers
- [ ] Verified archive: `tar -tzf backup.tar.gz`
- [ ] Prepared rollback plan

---

## Links

- 🐳 [Docker Volumes Docs](https://docs.docker.com/storage/volumes/)
- 🔄 [offen/docker-volume-backup](https://github.com/offen/docker-volume-backup)
- 📦 [Docker Compose Production](https://docs.docker.com/compose/production/)
