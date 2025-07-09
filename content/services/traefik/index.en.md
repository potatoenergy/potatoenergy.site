---
author: ["Potato Energy Team", "ponfertato"]
categories: ["Infrastructure", "Services"]
description: "Overview of a project's basic infrastructure stack"
slug: "traefik-errorpages-stack"
title: "Traffic and errors: the foundation of infrastructure"
---

### 1. Traefik: Smart Router [🛠️](https://traefik.io/)

**Purpose** 
Basic reverse proxy and inbound traffic controller with:
- Automatic SSL via Let's Encrypt
- Integration with Authelia for RBAC
- Load balancing between services

**Technical implementation**  
- Version: **v3.4.3** 
- Ports: 80 (HTTP), 443 (HTTPS)
- Network Policies: Via traefik network only
- Configuration: Static file + dynamic rules via Docker

**Security and Access**  
- Dashboard: `potatoenergy.ru/traefik` (only group `admin`)
- Certificates: Stored in encrypted `acme.json`
- Auditing: Real-time logging (LOG_LEVEL=debug)

**Features**.  
- HTTP/3 and QUIC support
- Automatic configuration update without downtime
- Integration with Prometheus for monitoring

---

### 2. Error-Pages: Custom Error Pages [🚨](https://github.com/tarampampam/error-pages)

**Purpose** 
Generate customized pages for:
- 4xx errors (client-side)
- 5xx errors (server)
- Maintenance

**Maintenance Implementation**  
- Version: **3.3.3** 
- Templates: Dynamic {status} substitution
- Configuration: Handle 400-599 statuses

**Security and Access**  
- Public Access: All users
- Management: Via PR to template repository
- Logic: Separate container with nginx

**Features**  
- Design theme support
- Automatic fallback to default pages
- Integration with Grafana for error analysis

---

### Example middleware configuration
```yaml
middlewares:
  error-pages:
    errors:
      query: /{status}.html
      service: error-pages
      status: 400-599
````
*Rules apply to all traffic through priority routers.

---

# Why is this important?  
This stack provides:
1. **Reliability** - automatic recovery from failures
2. **Single point of entry** for all services
3. **Flexible access control** through user groups
4. **Professional UX** even in emergency situations
