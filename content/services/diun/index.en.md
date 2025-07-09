---
author: ["Potato Energy Team", "ponfertato"]
categories: ["Security", "Infrastructure"]
description: "Automatic monitoring of Docker image updates"
slug: "diun-update-watcher"
title: "Diun: Container Relevance Guardian"
---

### Update Tracking System [🔄](https://github.com/crazy-max/diun)

**Purpose** 
Automatically detect new versions of images with:
- Discord/Telegram notifications
- Support for 15+ registries (Docker Hub, GHCR)
- 90 days of change history
- Portainer integration

**Technical implementation**  
- Version: **4.29.0** 
- Checking: Every 6 hours + random delay
- Storage: SQLite database with encryption
- Providers: Flexible configuration via YAML

**Security and Access**  
- Permissions: Limited User (UID 1000)
- Notifications: Only for `admin` and `dev` groups
- Audit: Full log of scans
- Isolation: Separate Docker network

**Features**  
- Custom message templates
- Ignore by tags/tags
- Support for Watchtower-compatible providers
- Export data to Prometheus

---

### Example notification
```yaml
template: |
  🚨 Обнаружено обновление для `{{ .Entry.Image }}`
  Текущая версия: {{ .Entry.CurrentTag }}
  Новая версия: {{ .Entry.NewTag }}
  Ссылка: {{ .Entry.Metadata.URL }}
```
*Sends to Discord and Telegram*

---

# Why is it important?
1. **Timely security updates**
2. **Monitoring changes** to dependencies
3. **Prevent drift** of configurations
4. **Automize** routine checks
