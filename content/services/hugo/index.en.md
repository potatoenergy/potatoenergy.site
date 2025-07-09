---
author: ["Potato Energy Team", "ponfertato"]
categories: ["Documentation", "Community"]
description: "Content publishing and collaboration platform"
slug: "hugo-remark42-stack"
title: "Hugo + Remark42: Knowledge and feedback"
---

### Static Website Generator [📖](https://gohugo.io/)

**Purpose** 
Quickly publish documentation and blogs with:
- Instant content updates
- Markdown support
- Automatic navigation building
- Preview drafts

**Technical implementation**  
- Version: **0.148.0** 
- Build: Development mode with live reboot
- Themes: Custom templates
- Integration: Automatic Deploy via Git

**Security and Access**  
- Editing: CI/CD only
- Publishing: Automatic after merge to main
- Change History: Git log for the entire period
- Backup: Hourly snapshots

**Features**  
- Support for shortcodes
- SEO optimization out of the box
- Multilanguage
- OpenGraph integration

---

### Comment System [💬](https://remark42.com/)

**Purpose** 
Decentralized platform for:
- Discussion of material
- Anonymous commenting
- Content moderation
- Feedback collection

**Technical implementation**  
- Version: **v1.14.0**
- Storage: Local volume
- Moderation: Web interface for admins
- Notifications: Discord + Telegram

**Security and Access**  
- Authentication: Telegram or anonymous
- Moderation: Group `admin`
- Data: GDPR compliant storage
- Backup: Daily backups

**Features**
- Votes and reactions
- Support for markdown in comments
- Automoderation via AI
- Data export to JSON

---

### Example integration
```html
<script>
  var remark_config = {
    host: 'https://potatoenergy.ru/remark42',
    site_id: 'potatoenergy',
    components: ['embed']
  };
</script>
```
*Added to the Hugo template automatically.

---

# Why is this important?
1. **Instant publishing** of changes
2. **Full control** over content
3. **Spam protection** through moderation
4. **Lossless knowledge archiving**

All content is available under [CC BY-SA 4.0](https://creativecommons.org/licenses/by-sa/4.0/). Use the [Issues system](https://codeberg.org/potatoenergy/potatoenergy.site) to suggest edits.
