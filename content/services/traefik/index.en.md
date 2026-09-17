---
author: ["Potato Energy Team", "ponfertato"]
categories: ["Infrastructure", "Services"]
description: "Overview of the project's basic infrastructure stack"
slug: "traefik"
title: "Traefik: Reverse Proxy and Routing"
---

### Traefik: Reverse Proxy and Routing [🛠️](https://traefik.io/)

**Reverse proxy** that routes requests to the required services.

**Functions:**

- 🔐 Automatic issue and renewal of SSL certificates (Let's Encrypt)
- 🚦 Domain-based routing: `grafana.*/` → Grafana, `cloud.*/` → Nextcloud
- ⚡ Service updates without connection drops
- 🧩 Middleware: authentication (Authelia), rate-limiting, redirects, compression
- 📊 Metric export for Prometheus - real-time traffic visibility

**How it works:**

1. `service.potatoenergy.ru` is entered in the browser
2. Traefik checks the rules, applies SSL and middleware
3. The request is routed to the correct container

**For administrators:**
Dynamic configuration via Docker labels and configuration files, hot-reload without restart, Docker integration.

**Access:**
automatically • the basis for routing all Potato Energy services
