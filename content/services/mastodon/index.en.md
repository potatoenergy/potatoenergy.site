---
author: ["Potato Energy Team", "ponfertato"]
categories: ["Social Network", "Federation"]
description: "Decentralized social platform powered by ActivityPub"
slug: "mastodon-social"
title: "Mastodon: Your Federation Space"
---

### A new type of social network [🌐](https://joinmastodon.org/)

**Purpose** 
An open platform for:
- Publishing posts and media
- Interacting with federated instances
- Creating communities (instances)
- Content moderation

**Technical implementation**  
- Version: **v4.4.0**
- Database: PostgreSQL 15
- Task Queues: Redis 8.0.3 + Sidekiq
- Architecture: Microservice (5 components)
- Localization: Full Russian support

**Security and Access**  
- Authentication: OIDC via Authelia
- Encryption: E2E for private messages
- Moderation: Tools for `admin` and `moderator` groups
- Audit: 12 months log storage

**Features**  
- ActivityPub support
- Custom emoji and themes
- Advanced privacy settings
- Prometheus integration for monitoring

---

### Example federation
```yaml
services:
  web: # HTTP request processing
  streaming: # Real-time updates
  sidekiq: # Background tasks
  postgres: # Data warehouse
  redis: # Session caching
```
# Handles 500+ transactions per second #

---

# Why is it important?
1. **Data control** without centralized platforms
2. **Compatibility** with Fediverse (PeerTube, Pixelfed)
3. **Flexible moderation** of content
4. **Transparent** feed algorithms

Join our community via [social.potatoenergy.ru](https://social.potatoenergy.ru). All publications are protected by [Creative Commons BY-SA 4.0](https://creativecommons.org/licenses/by-sa/4.0/).
