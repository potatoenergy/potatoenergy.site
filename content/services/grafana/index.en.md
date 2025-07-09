---
author: ["Potato Energy Team", "ponfertato"]
categories: ["Visualization", "Analytics"]
description: "Centralized platform for metrics analysis"
slug: "grafana-dashboards"
title: "Grafana: Every Bit Visibility"
---

### Monitoring Dashboard [📈](https://grafana.com/)

**Purpose** 
Single space for:
- Prometheus metrics visualization
- Analyze infrastructure logs
- Create custom dashboards
- Collaboration with data

**Technical implementation**  
- Version: **OSS 11.6.0**
- Data Sources: 15+ integrations
- Storage: Local volume with backups
- Authorization: OAuth2 via Authelia
- Roles: Dynamic assignment via groups

**Security and Access**  
- Anonymous access: Viewer only
- Editing: Group `dev` (Editor)
- Administration: Group `admin`
- Sessions: JWT with 12-hour TTL

**Features**.  
- 50+ preset dashboards
- SQL-like queries for metrics
- Report export to PDF
- Event annotation system

---

### Ecosystem Integration
```yaml
auth:
  oauth:
    role_mapping: |
      contains(groups[*], 'admin') ? 'Admin' : 
      contains(groups[*], 'dev') ? 'Editor' : 
      'Viewer'
```
*Automatic synchronization of permissions with Authelia*

---

# Why is it important?
1. **Single interface** for all metrics
2. **Flexible alerts** with linking to dashboards
3. **Cross-platform analysis** (Prometheus/Loki/MySQL)
4. **Collaboration** via shared snapshots

Example Usage: [Service Availability](https://potatoenergy.ru/grafana/d/cadvisor-exporter/container-monitoring). To create personalized dashboards, please contact members of the `dev` group.
