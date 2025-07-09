---
author: ["Potato Energy Team", "ponfertato"]
categories: ["DevOps", "Governance"]
description: "Centralized container management with Git integration"
slug: "portainer-gitops"
title: "Portainer: conductor of the container orchestra"
---

### Portainer CE: Visual Infrastructure Control [🎛️](https://www.portainer.io/)

**Purpose** 
Docker environment control panel with:
- GitOps integration (Codeberg)
- Visual compose file editor
- Service state monitoring

**Technical implementation**  
- Version: **2.30.1-alpine**
- Git synchronization: Autodeploy from repository
- Data storage: Local volume
- API: Integration via Docker socket

**Security and Access**  
- Administration: `potatoenergy.ru/portainer` (group `admin`)
- RBAC: Authelia based role model
- Audit: History of all changes for 90 days

**Features**  
- GitOps automation: push events → deploy
- Templates for fast deployment
- Web terminal for emergency access
- Problem notification system

---

### GitOps in action [🔄](https://about.gitlab.com/topics/gitops/)
The principle of "infrastructure as code" is implemented through:
1. Centralized repository on Codeberg
2. automatic synchronization of changes
3. Verification of changes via MR
4. git revert

---

# Why is it important?
1. **Version control** for the entire infrastructure
2. **Transparency of changes** through git history
3. **Reduce manual errors** when deploying
4. **Cross Audience** who/what/when changed what/when
