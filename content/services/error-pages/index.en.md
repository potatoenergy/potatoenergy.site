---
author: ["Potato Energy Team", "ponfertato"]
categories: ["Infrastructure", "Services"]
description: "Customized error page generation"
slug: "error-pages"
title: "Error-Pages: Custom Error Pages"
---

### Error-Pages: Custom error pages [🚨](https://github.com/tarampampam/error-pages)

**Friendly messages** instead of the standard "404 Not Found".

**What does:**

- 🎨 Stylish pages for 4xx/5xx errors in a single Potato Energy brand
- 🌍 Automatic language detection and dark/light theme
- 🔧 Useful tips: "check the URL", "go back to the main page", "write to support"
- 📊 Error logging for administrators
- ⚡ Lightweight static pages - load even in case of backend failures

**How it works:**

1. If there is an access error, Nginx/Traefik redirects to a custom page.
2. You see a clear message with options for action.
3. The administrator receives notification of critical failures

**For administrators:**
Flexible customization of texts, redirects, and styles through a single configuration.

**Access:** automatically • triggered in case of errors
