---
author: ["Potato Energy Team", "ponfertato"]
categories: ["android", "shizuku", "guide"]
date: "2026-03-20T13:15:00+03:00"
description: "Shizuku: Android system capabilities without root. Setup and examples with Obtainium and SAI."
draft: false
series: ["Android Tips"]
slug: "shizuku-guide-en"
tags: ["android", "shizuku", "obtainium", "sai", "adb", "system"]
title: "Shizuku: Android System Access Without Root"
---

Shizuku is a service that gives apps access to Android system APIs **without root**. It works via ADB (Android Debug Bridge), using `shell` privileges.

**Why you need it**:

- Install apps without confirmation (Obtainium, SAI)
- Freeze/unfreeze apps (Ice Box, Hail)
- Manage permissions (AppOps, Permission Pilot)
- Change system settings (DarQ, Naptime)
- Remove system apps (Canta, AppManager)

> 💡 Shizuku doesn't grant full root - only limited access to system functions. Safer than root, but more powerful than a regular app.

---

## 📦 Installation and setup

### Step 1: Download Shizuku

- [F-Droid](https://f-droid.org/packages/moe.shizuku.privileged.api/) (recommended)
- [GitHub Releases](https://github.com/RikkaApps/Shizuku/releases)

### Step 2: Enable Developer Mode

1. Settings → About phone → Tap "MIUI version" / "Build number" 7 times
2. Go back to Settings → Additional settings → **Developer options**

### Step 3: Enable Wireless Debugging

1. In Developer options → ✅ **Wireless debugging**
2. Tap "Pair device with pairing code" → note code and port
3. In Shizuku: "Start" → "Pairing" → enter code and port
4. After pairing: "Start" → service will launch

**Check status**:

```
Status: Running
Version: 13.x.x
```

> ⚠️ After phone reboot, Shizuku must be restarted manually (process doesn't persist).

---

## 🔧 Example 1: Obtainium + Shizuku

[Obtainium](https://github.com/ImranR98/Obtainium) - app update manager directly from sources (GitHub, GitLab, F-Droid).

### Why Shizuku for Obtainium

| Without Shizuku             | With Shizuku           |
| --------------------------- | ---------------------- |
| Manual install confirmation | Automatic installation |
| Doesn't work with split-APK | Supports all formats   |
| Requires "Unknown sources"  | Installs via system PM |

### Setup

1. Install [Obtainium from F-Droid](https://f-droid.org/packages/dev.imranr.obtainium.fdroid/)
2. Open Obtainium → Settings → **Installation method**
3. Select **Shizuku** (auto-detected if service is running)
4. Add apps to track:
   - Enter repo URL: `https://github.com/user/repo`
   - Or browse catalog
5. On update: Obtainium downloads → installs via Shizuku → no confirmation needed

**Example app entry**:

```
Source: GitHub
URL: https://github.com/RikkaApps/Shizuku
Filter: Releases (stable)
Format: APK
```

---

## 🔧 Example 2: SAI + Shizuku

[SAI (Split APKs Installer)](https://github.com/Aefyr/SAI) - installer for split-APK, XAPK, APKS (formats not supported by default installer).

### Why Shizuku for SAI

| Without Shizuku                  | With Shizuku                       |
| -------------------------------- | ---------------------------------- |
| Manual confirmation for each APK | Batch install without confirmation |
| Doesn't work with some formats   | Supports all split formats         |
| Installation errors              | Reliable install via system PM     |

### Setup

1. Install [SAI from F-Droid](https://f-droid.org/packages/com.aefyr.sai.fdroid/)
2. Open SAI → Settings → **Installation method**
3. Select **Shizuku** (or "Session API + Shizuku" for max compatibility)
4. Install app:
   - Tap "Install APK" → select `.xapk`, `.apks`, `.apk` file
   - SAI unpacks → installs via Shizuku → done

**Supported formats**:

- `.apk` - standard package
- `.xapk` - APK + OBB data
- `.apks` / `.apk-m` - split-APK (multiple files for different architectures)

---

## 🔄 Auto-start Shizuku (optional)

After reboot, Shizuku stops. Auto-start options:

### Option 1: Tasker + ADB (no root)

```bash
# Script for Tasker: start-shizuku.sh
adb shell sh /sdcard/Android/data/moe.shizuku.privileged.api/start.sh
```

**Setup**:

1. Install [Tasker](https://play.google.com/store/apps/details?id=net.dinglisch.android.taskerm)
2. Create task → "Run Shell" → command above
3. Trigger: "Device Boot"

### Option 2: KernelSU / Magisk (with root)

If rooted - install Shizuku as system app:

```bash
adb push shizuku.apk /data/local/tmp/
adb shell su -c "pm install /data/local/tmp/shizuku.apk"
adb shell su -c "sh /sdcard/Android/data/moe.shizuku.privileged.api/start.sh"
```

**Benefit**: Shizuku starts automatically on boot.

---

## 🔍 Diagnostics

```bash
# Check if Shizuku is running
adb shell sh /sdcard/Android/data/moe.shizuku.privileged.api/start.sh --check

# View Shizuku logs
adb logcat | grep -i shizuku

# Check app access to Shizuku
adb shell dumpsys package moe.shizuku.privileged.api | grep -A5 "Granted permissions"
```

---

## ⚠️ Common issues

```bash
# Shizuku won't start
→ Toggle wireless debugging: off → on
→ Reboot phone and restart Shizuku
→ Check if antivirus/optimizer is blocking it

# Obtainium/SAI don't detect Shizuku
→ Ensure service is running (status "Running" in app)
→ Restart Shizuku and target app
→ Check permissions: Settings → Apps → [App] → Permissions

# "Package parser error" on install
→ File corrupted - re-download
→ Unsupported format - check SAI version
→ Not enough space - clear cache

# Shizuku stops by itself
→ Settings → Battery → [Shizuku] → ✅ No restrictions
→ Settings → Apps → [Shizuku] → ✅ Auto-start
```

---

## 🛡 Security

### What can an app do with Shizuku access

| Action                | Risk                                                   |
| --------------------- | ------------------------------------------------------ |
| Install/uninstall app | Medium (requires user confirmation in Obtainium/SAI)   |
| Change permissions    | Medium (only for own package or with explicit consent) |
| Read logs             | Low (only own logs)                                    |
| Access files          | Low (only with explicit permission)                    |

### How to minimize risks

1. Install Shizuku only from [F-Droid](https://f-droid.org/) or [GitHub](https://github.com/RikkaApps/Shizuku)
2. Grant Shizuku access only to trusted apps (Obtainium, SAI, AppManager)
3. Stop Shizuku when not in use
4. Don't enable wireless debugging on public networks

---

## Links

- 🔧 [Shizuku on F-Droid](https://f-droid.org/packages/moe.shizuku.privileged.api/)
- 📦 [Obtainium on F-Droid](https://f-droid.org/packages/dev.imranr.obtainium.fdroid/)
- 📱 [SAI on F-Droid](https://f-droid.org/packages/com.aefyr.sai.fdroid/)
- 📘 [Shizuku Documentation](https://shizuku.rikka.app/)
- 🔐 [Android ADB Docs](https://developer.android.com/tools/adb)
