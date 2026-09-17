---
author: ["Potato Energy Team", "ponfertato"]
categories: ["Infrastructure", "Services"]
description: "Customized error page generation"
slug: "error-pages"
title: "Error-Pages: Custom Error Pages"
---

### Error-Pages: Custom Error Pages [🚨](https://github.com/tarampampam/error-pages)

**Custom pages** instead of the standard "404 Not Found".

**Functions:**

- 🎨 Pages for 4xx/5xx errors in a unified Potato Energy style
- 🌍 Automatic language detection and dark/light theme
- 🔧 Tips: "check the URL", "go back to the main page", "contact support"
- 📊 Error logging for administrators
- ⚡ Static pages - load even during backend failures

**How it works:**

1. On access error, Nginx/Traefik redirects to a custom page
2. A message with action options is displayed
3. The administrator receives notification of critical failures

**For administrators:**
Customization of texts, redirects, and styles through a single configuration.

**Access:**
automatically • triggered on errors
