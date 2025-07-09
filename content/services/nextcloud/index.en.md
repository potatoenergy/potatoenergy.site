---
author: ["Potato Energy Team", "ponfertato"]
categories: ["Storage", "Collaboration"]
description: "Secure cloud storage with advanced features"
slug: "nextcloud-storage"
title: "Nextcloud: Your digital workspace"
---

### Nextcloud Platform [☁️](https://nextcloud.com/)

**Purpose** 
Secure space for:
- File storage and synchronization
- Collaborate on documents
- Video conferencing via Spreed
- Manage tasks and calendars

**Technical Implementation**  
- Version: **31.0.6** 
- Database: PostgreSQL 15
- Caching: Redis 8.0.3
- Architecture: Microservice (6 components)
- Monitoring: Export metrics to Prometheus

**Security and Access**  
- Authentication: OIDC via Authelia
- Encryption: Server-side AES-256
- Roles: 
  - `pro` group - advanced storage
  - All users - basic access
- Auditing: Logging of all transactions

**Features**  
- Automatic image optimization
- Integration with OnlyOffice
- WebDAV/CalDAV support
- Quota system for users

---

### Example architecture
```yaml
services:
 postgres: # Metadata store
 redis: # Session cache
 nextcloud: # Platform core
 nginx: # Request processing
 cron: # Background tasks
 exporter: # Monitoring
```
# Handles 1000+ requests per minute #

---

# Why is it important?
1. **Total control** over your data
2. **Compatibility** with any device.
3. **Legal compliance** with GDPR
4. **Uninterrupted operation** thanks to clustering

Please contact your administrator to activate pro features. All data is protected by [end-to-end encryption](https://nextcloud.com/endtoend/) and backed up daily.
