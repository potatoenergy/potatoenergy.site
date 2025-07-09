---
author: ["Potato Energy Team", "ponfertato"]
categories: ["AI", "Productivity"]
description: "Smart Assistant with Ecosystem Integration"
slug: "open-webui-assistant"
title: "Open WebUI: Your Intelligent Assistant"
---

### Next Generation AI Platform [🤖](https://github.com/open-webui/open-webui)

**Purpose** 
Personalized AI assistant with:
- Russian language text generation
- Real-time information retrieval
- Document and media content analysis
- Integration with external APIs

**Technical implementation**  
- Version: **v0.6.15** 
- Core: RAG architecture with web search
- Caching: Redis 8.0.3
- Localization: Full Russian support
- Monitoring: OpenTelemetry + Prometheus

**Security and Access**  
- Authentication: OAuth2 via Authelia
- Roles: 
  - Admins (`admin` group) - full access
  - Users - basic functions
- Data: AES-256 storage encryption

**Features**  
- Search via Google Programmable Search
- YouTube video processing in native language
- Export dialogs to Markdown/PDF
- Web Hook System for Integrations

---

### Example search query
`` ``python
async def web_search(query: str):
    return await google_pse_search(
        engine_id="a6fd1036793ad45e8",
        api_key="...",
        query=query,
        lang="ru"
    )
`` ``
*Available for members of the `dev` group.

---

# Why is it important?
1. **Contextual understanding** of queries in Russian
2. **Secure storage** of dialog history
3. **Automatic updating** of knowledge base
4. **Integration** with the team's working tools

To get started, visit [chat.potatoenergy.ru](https://chat.potatoenergy.ru). Request access to advanced features from the administrators.
