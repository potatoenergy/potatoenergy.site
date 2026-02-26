---
author: ["Potato Energy Team", "ponfertato"]
categories: ["Infrastructure", "Services"]
description: "Overview of the project's basic infrastructure stack"
slug: "traefik"
title: "Traefik: Smart Router"
---

### Traefik: Smart Router [🛠️](https://traefik.io/)

**Your digital dispatcher**, who directs requests to the necessary services.

**What does:**

- 🔐 Auto-issue and update of SSL certificates (Let's Encrypt)
- 🚦 Routing by domains: `grafana.*/` → Grafana, `cloud.*/` → Nextcloud
- ⚡ Zero-downtime deployment: service updates without disconnecting connections
- 🧩 Middleware: authentication (Authelia), rate-limiting, redirects, compression
- 📊 Exporting metrics for Prometheus - real - time traffic visibility

**How it works:**

1. You enter `service.potatoenergy.ru` in the browser
2. Traefik checks the rules, applies SSL and middleware
3. The request gets into the right container - you see the service

**For administrators:**
Dynamic configuration via Docker labels and configuration files, hot-reload without restarting, integration with Docker.

**Access:** automatically • the basis for routing all Potato Energy services
