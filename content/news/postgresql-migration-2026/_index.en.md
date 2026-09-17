---
author: ["Potato Energy Team", "ponfertato"]
categories: ["Updates", "Infrastructure"]
date: "2026-04-22T18:00:00+03:00"
description: "Planned database migration completed: Authelia and Mastodon moved to PostgreSQL 18, Nextcloud to PostgreSQL 17."
slug: "postgresql-migration-2026"
title: "PostgreSQL Migration: Authelia & Mastodon to 18.x, Nextcloud to 17.x"
---

The planned database migration for the Potato Energy project is complete. The transition was performed without data loss and with minimal downtime.

### What changed

| Service       | Before          | After               | Reason                                                                      |
| ------------- | --------------- | ------------------- | --------------------------------------------------------------------------- |
| **Authelia**  | PostgreSQL 15.x | **PostgreSQL 18.x** | Required for new authentication features and improved security              |
| **Mastodon**  | PostgreSQL 15.x | **PostgreSQL 18.x** | Support for modern ActivityPub versions and better federation performance   |
| **Nextcloud** | PostgreSQL 15.x | **PostgreSQL 17.x** | Stable branch with long-term support and compatibility with current plugins |

> 💡 PostgreSQL 18 was released on September 25, 2025. It brings improved I/O performance, enhanced partitioning capabilities, and an updated query planner.

### How the migration was performed

1. **Logical backup** — `pg_dumpall` for all databases
2. **Physical volume archive** — a safety copy of raw data
3. **Deployment of new containers** — with updated images
4. **Data restoration** — `psql < pg_dumpall.sql`
5. **Verification** — checking versions, paths, and table integrity

The full process is described in the technical article: [PostgreSQL in Docker: Major Version Migration Without Data Loss](/en/blog/infrastructure/docker/postgres-upgrade/)
