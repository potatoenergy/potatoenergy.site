---
author: ["Potato Energy Team", "ponfertato"]
categories: ["android", "hyperos", "guide"]
date: "2026-03-20T13:25:00+03:00"
description: "Remove Google Chrome from second space on HyperOS (Xiaomi/POCO). Solutions via Shizuku, AppManager, and ADB."
draft: false
series: ["Android Tips"]
slug: "chrome-dual-fix"
tags: ["android", "hyperos", "chrome", "shizuku", "appmanager", "dual-apps"]
title: "HyperOS: Remove Chrome from Second Space"
---

## The Problem

After switching to HyperOS (POCO, Xiaomi), links from apps (Telegram, WhatsApp, etc.) open in the cloned Chrome account instead of the main one - even when the main Chrome is set as default browser.

**Symptoms**:

- Click link → Chrome opens in second space
- Default browser settings show main Chrome
- Resetting settings doesn't help

**Cause**: HyperOS prioritizes cloned apps when handling intents, ignoring user preference.

> 💡 Issue reproduces on MIUI 14 / HyperOS 1.0+ with "Dual Apps" / "Second Space" enabled.

---

## ❌ Why simple fixes don't work

### ADB removal (temporary)

```bash
adb shell
pm uninstall -k --user 999 com.android.chrome
```

**Problem**: after reboot, Chrome in second space is **quietly reinstalled**. `user 999` is the cloned profile ID, but HyperOS restores system apps on startup.

### Disable "Dual Apps"

**Downside**: removes **all** cloned apps and their data. Not suitable if you need other duplicates (messengers, banking apps).

---

## ✅ Solutions (by preference)

### Solution 1: Shizuku + AppManager (recommended)

Remove Chrome from second space **without root**, keeping other cloned apps intact.

#### Step 1: Install Shizuku

1. Download [Shizuku from F-Droid](https://f-droid.org/packages/moe.shizuku.privileged.api/)
2. Launch app → "Start via wireless debugging"
3. Enable wireless debugging in Developer Options:
   - Settings → About phone → Tap "OS version" 7 times
   - Settings → Additional settings → Developer options → ✅ Wireless debugging
4. Follow Shizuku instructions to pair

#### Step 2: Install AppManager

1. Download [AppManager from F-Droid](https://f-droid.org/packages/io.github.muntashirakon.AppManager/)
2. Open app → grant access via Shizuku (automatic)

#### Step 3: Remove Chrome from second space

1. Find `com.android.chrome` (Google Chrome)
2. Open app card → ⋮ → **Uninstall for user**
3. Confirm removal

**Result**: Chrome removed only from second space, main account unaffected.

#### Step 4: Verify

```bash
# Via ADB (optional)
adb shell pm list packages --user 999 | grep chrome
# Should be empty
```

---

### Solution 2: Change default browser

If you don't want to set up Shizuku:

1. Install alternative browser: [Chrome Beta](https://play.google.com/store/apps/details?id=com.chrome.beta), Firefox, Brave
2. Export bookmarks from main Chrome (sync with account)
3. Clear main Chrome data (optional)
4. Set new browser as default:
   - Settings → Apps → Default apps → Browser
5. Test link opening

**Pro**: no root, Shizuku, or ADB required  
**Con**: need to adapt to new browser

---

### Solution 3: ADB script with auto-launch (advanced)

If Shizuku doesn't work, automate ADB commands:

```bash
#!/bin/bash
# remove-chrome-clone.sh
adb wait-for-device
adb shell pm uninstall -k --user 999 com.android.chrome
echo "Chrome removed from second space"
```

**Auto-launch via Termux + ADB**:

1. Install [Termux](https://f-droid.org/packages/com.termux/)
2. Install [ADB Keyboard](https://f-droid.org/packages/de.stefanbechtold.simpleterm/) or use `adb tcpip`
3. Run script on boot via `~/.termux/boot/`

**Limitation**: after phone reboot, script must be run manually (or set up auto-launch via Tasker).

---

## 🔍 Diagnostics

```bash
# Check if Chrome is installed in second space
adb shell pm list packages --user 999 | grep chrome

# Check default browser
adb shell dumpsys package preferred | grep browser

# View intent handlers for links
adb shell dumpsys activity preferred-activities | grep -A5 "http"
```

---

## ⚠️ Common issues

```bash
# Shizuku won't start
→ Re-enable wireless debugging and re-pair
→ Restart Shizuku after system update

# AppManager doesn't see user 999
→ Grant permissions via Shizuku: Settings → Apps → AppManager → Permissions
→ Restart AppManager

# Chrome reinstalls after reboot
→ This is HyperOS behavior. Workaround: add script to auto-start (Termux + Tasker)
→ Or use Shizuku + AppManager with re-removal on boot (requires root)

# Links still open in clone
→ Clear default settings: Settings → Apps → Manage apps → ⋮ → Reset defaults
→ Reboot phone
```

---

## Links

- 🔧 [Shizuku on F-Droid](https://f-droid.org/packages/moe.shizuku.privileged.api/)
- 📱 [AppManager on F-Droid](https://f-droid.org/packages/io.github.muntashirakon.AppManager/)
- 🤖 [Termux on F-Droid](https://f-droid.org/packages/com.termux/)
- 📘 [Android Package Manager Docs](https://developer.android.com/tools/adb#pm)
