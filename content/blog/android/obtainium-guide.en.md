---
author: ["Potato Energy Team", "ponfertato"]
categories: ["android", "obtainium", "guide"]
date: "2026-03-20T13:18:00+03:00"
description: "Obtainium: automatic app updates directly from source repositories. Setup, filters, Shizuku integration."
draft: false
series: ["Android Tips"]
slug: "obtainium-guide"
tags: ["android", "obtainium", "updates", "github", "shizuku", "fdroid"]
title: "Obtainium: App Updates from Source"
---

Obtainium is an Android update manager that downloads apps **directly from developer repositories** (GitHub, GitLab, Codeberg, F-Droid), bypassing third-party stores.

**Why you need it**:

- Get updates faster than Google Play / F-Droid
- Avoid trackers and ads from third-party stores
- Control which versions are installed (stable, beta, pre-release)
- Automate updates without manual confirmation

> 💡 Obtainium doesn't host apps - it only points where to download them. You always know the source.

---

## 📦 Installation

### Download

- [F-Droid (recommended)](https://f-droid.org/packages/dev.imranr.obtainium.fdroid/)
- [GitHub Releases](https://github.com/ImranR98/Obtainium/releases)

### First-run setup

1. Open Obtainium → allow "Install unknown apps" (for Obtainium only)
2. Settings → **Installation method** → select **Shizuku** (if configured) or **System installer**
3. Settings → **Update check interval** → set frequency (recommended: 6–24 hours)

**Optional**:

- ✅ Show notifications for new versions
- ✅ Auto-download updates (requires stable internet)
- ✅ Auto-install (requires Shizuku)

---

## 🔍 Adding apps

### Method 1: By repository URL

```
Source: GitHub
URL: https://github.com/RikkaApps/Shizuku
Filter: Releases → Stable only
Format: APK (universal) or arm64-v8a (for performance)
```

**Steps**:

1. In Obtainium: "+" → "Add app"
2. Paste repository URL
3. Tap "Check" - Obtainium shows available versions
4. Configure filters (tags, pre-releases, architecture)
5. Save

### Method 2: From catalog

Obtainium has a built-in catalog of popular apps:

- Menu → "Catalog" → select app → "Add"
- Filters applied automatically

### Method 3: Import list

```json
// backup.json - exported config
{
  "apps": [
    {
      "sourceId": "github",
      "url": "https://github.com/user/repo",
      "includePrereleases": false,
      "filterReleaseTitlesByRegEx": "",
      "filterReleaseNotesByRegEx": "",
      "versionExtractionRegEx": "",
      "apkFilterRegEx": "arm64-v8a",
      "invertAPKFilter": false
    }
  ]
}
```

**Import**: Settings → "Backup" → "Restore"

---

## ⚙️ Advanced settings

### Version filters

| Parameter       | Example       | Description                        |
| --------------- | ------------- | ---------------------------------- |
| `Stable only`   | ✅            | Ignore beta, alpha, rc             |
| `Regex (title)` | `^v[0-9.]+$`  | Accept only versions like `v1.2.3` |
| `Regex (notes)` | `(?i)android` | Search keywords in release notes   |

### APK filters

| Parameter        | Example          | Purpose                          |
| ---------------- | ---------------- | -------------------------------- |
| `Filter by name` | `arm64-v8a`      | Download only for 64-bit devices |
| `Invert filter`  | ✅               | Exclude specific architectures   |
| `Min size`       | `1000000` (1 MB) | Filter out empty/corrupted files |

### Notifications & auto-update

```
Settings → Notifications:
✅ Show on new version
✅ Sound / Vibration (optional)

Settings → Auto-update:
✅ Enable (requires Shizuku for silent install)
⏰ Interval: 6 hours
🌙 Only when charging + WiFi (recommended)
```

---

## 🔗 Shizuku integration

### Why Shizuku for Obtainium

| Without Shizuku             | With Shizuku                        |
| --------------------------- | ----------------------------------- |
| Manual install confirmation | Fully automatic installation        |
| Doesn't work with split-APK | Supports all formats                |
| Requires "Unknown sources"  | Installs via system Package Manager |

### Setup

1. Ensure Shizuku is running (status "Running")
2. In Obtainium: Settings → **Installation method** → **Shizuku**
3. Grant access on first install attempt
4. Test: update any app

**Check logs** (if something fails):

```bash
adb logcat | grep -i obtainium
adb logcat | grep -i shizuku
```

---

## 📊 Comparison with alternatives

| Manager          | Sources                                         | Auto-install      | Split-APK  | Privacy   |
| ---------------- | ----------------------------------------------- | ----------------- | ---------- | --------- |
| **Obtainium**    | GitHub, GitLab, Codeberg, F-Droid, direct links | ✅ (with Shizuku) | ✅         | 🔒 High   |
| **F-Droid**      | F-Droid repos only                              | ❌                | ❌         | 🔒 High   |
| **Aurora Store** | Google Play (anonymously)                       | ❌                | ✅         | 🔐 Medium |
| **APKUpdater**   | GitHub, F-Droid, APKMirror                      | ❌                | ⚠️ Partial | 🔐 Medium |
| **Google Play**  | Play Store only                                 | ✅                | ✅         | 🔓 Low    |

**When to choose Obtainium**:

- You trust developers directly
- You want updates faster than official stores
- You need split-APK support and flexible filters
- Privacy and source control matter to you

---

## ⚠️ Common issues

```bash
# "Failed to fetch version info"
→ Check internet connection
→ Ensure repo is public (or add token in settings)
→ Try "Check manually" in app card

# "Installation canceled by user"
→ Without Shizuku: expected - confirm install manually
→ With Shizuku: check if service is running and permissions granted

# "Unsupported APK format"
→ Enable "Support split-APK" in settings
→ Update Obtainium to latest version
→ Use SAI as fallback installer

# "App not updating despite newer version"
→ Check filters: new version may be marked as pre-release
→ Clear app cache: Settings → Apps → Obtainium → Storage → Clear cache
```

---

## 🛡 Security

### How Obtainium ensures security

| Mechanism                  | Description                                                  |
| -------------------------- | ------------------------------------------------------------ |
| **Direct download**        | No proxies - file downloads straight from developer's server |
| **Signature verification** | On update, new APK signature is compared with installed one  |
| **Open source**            | Obtainium code is auditable on GitHub                        |
| **No telemetry**           | App doesn't collect usage data                               |

### Security recommendations

1. Add only trusted repositories (official developer accounts)
2. Enable "Verify signature on update" in settings
3. Use Shizuku only with trusted apps
4. Keep Obtainium itself updated

---

## Links

- 📦 [Obtainium on F-Droid](https://f-droid.org/packages/dev.imranr.obtainium.fdroid/)
- 🐙 [Obtainium on GitHub](https://github.com/ImranR98/Obtainium)
- 🔧 [Shizuku on F-Droid](https://f-droid.org/packages/moe.shizuku.privileged.api/)
- 📘 [Obtainium Wiki](https://github.com/ImranR98/Obtainium/wiki)
- 🔍 [Regex tutorial](https://regex101.com/)
