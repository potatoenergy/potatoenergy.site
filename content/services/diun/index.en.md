---
author: ["Potato Energy Team", "ponfertato"]
categories: ["Security", "Infrastructure"]
description: "Automatic monitoring of Docker image updates"
slug: "diun"
title: "Diun: Docker Image Update Monitoring"
---

### Diun: Docker Image Update Monitoring [🔄](https://github.com/crazy-max/diun)

**Notification service** for Docker image updates.

**Functions:**

- 🔍 Monitors image tags in Docker Hub, GitHub, and private registries
- 📩 Sends notifications to Telegram, Discord when new versions are released
- 📋 Maintains a detailed log of changes with links to releases

**Usage:**

1. Diun works in the background - no configuration required
2. A notification arrives when an update is available for the service
3. The administrator applies the update or sets up auto-update

**For administrators:**
Rules, filtering, notifications - in the config.
