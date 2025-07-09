---
author: ["Potato Energy Team", "ponfertato"]
categories: ["Security", "Infrastructure"]
description: "Centralized authentication and authorization system"
slug: "authelia-security-stack"
title: "Authelia: protecting the digital space"
---

### 1. Authelia: Single Sign-on and Access Control [🔒](https://www.authelia.com/)

**Purpose** 
Centralized authentication system with:
- Two Factor Authentication (2FA)
- Rights management via user groups
- Integration with 15+ services via OIDC

**Technical implementation**  
- Version: **4.39.4**
- Data Storage: PostgreSQL 15
- Caching: Redis 8.0.3
- Secrets: Key encryption via Docker Secrets
- Monitoring: Export metrics to Prometheus

**Security and Access**  
- Administration: `admin` group only
- Access Policies: Cascading rules in authelia.configuration.yml
- Auditing: Logging of all login attempts
- Backup: Hourly database backups

**Features**  
- U2F/WebAuthn support
- Integration with Traefik ForwardAuth
- Automatic bruteforce lockout
- Custom login pages

---

### 2. Auxiliary components

**PostgreSQL**  
- Encryption algorithm: AES-256 (STORAGE_ENCRYPTION_KEY)
- Healthcheck: Check availability every 5 sec
- Volume: Local data storage

**Redis**  
- Authentication: Password from Docker Secrets
- Protocol: RESP3 with TLS
- Load: Up to 10,000 sessions/sec

---

### Example OIDC Integration
````yaml
identity_providers:
  oidc:
    clients:
      - client_id: nextcloud
        client_secret: $argon2id
        redirect_uris:
          - https://cloud.potatoenergy.ru/apps/user_oidc/code
```
*Works for Grafana, Portainer and Mastodon*

---

# Why is this important?
1. **Single account** for all ecosystem services
2. **Granular control** through user groups
3. **Compliance** with GDPR and ISO 27001
4. **Transparency** - full audit of actions

To configure 2FA or change access rights, contact your administrator via [helpdesk](mailto:mail@potatoenergy.ru).
