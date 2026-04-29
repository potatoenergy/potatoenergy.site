---
author: ["Potato Energy Team", "ponfertato"]
categories: ["hardware", "orangepi", "bluetooth", "guide"]
date: "2026-04-29T12:40:00+03:00"
description: "Setting up Bluetooth on Orange Pi 3B: Spreadtrum UWE5622 chip, hciattach, BlueZ. Fixing 'No default controller' and 'Error.Busy'."
draft: false
series: ["Hardware"]
slug: "orangepi3b-bluetooth"
tags: ["orangepi", "bluetooth", "spreadtrum", "bluez", "hciattach", "linux"]
title: "Orange Pi 3B: Enabling Bluetooth (Spreadtrum UWE5622)"
---

On Orange Pi 3B, the built-in Bluetooth chip **Spreadtrum UWE5622** is connected via UART (`/dev/ttyBT0`). Unlike USB adapters, it requires:

1. Loading firmware and calibration data before initialization
2. Running `hciattach_opi` with correct flags
3. Correct startup order: chip initialization first, then BlueZ daemon

**Symptoms**:

- `bluetoothctl scan on` → `No default controller available`
- `btmgmt info` → `Index list with 0 items`
- `hciconfig -a` shows `hci0`, but `bluetoothctl` doesn't see it
- Error `org.bluez.Error.Busy` when trying to power on

**Cause**: The service `orangepi3b-sprd-bluetooth.service` runs `hciattach_opi` with the `-n` flag (no-detach), which holds the device and prevents BlueZ from registering the controller.

---

## ✅ Solution: Two Steps

### Step 1: Install Missing Utilities

```bash
sudo apt update
sudo apt install -y rfkill bluez
```

> `rfkill` is needed to unblock the adapter, `bluez` is the Bluetooth stack itself.

### Step 2: Fix the Initialization Service

#### Option A: Quick Fix (Manual)

```bash
# 1. Stop everything
sudo systemctl stop orangepi3b-sprd-bluetooth
sudo systemctl stop bluetooth
sudo killall -9 hciattach_opi 2>/dev/null
sleep 2

# 2. Load drivers and unblock adapter
sudo modprobe -a sprdbt_tty sprdwl_ng
sudo rfkill unblock all

# 3. Run initialization WITHOUT -n flag (hciattach will detach automatically)
sudo /usr/bin/hciattach_opi -s 1500000 /dev/ttyBT0 sprd
# ← Process will complete in 2-3 seconds if chip responded

# 4. Start BlueZ - it will pick up the ready hci0
sudo systemctl start bluetooth
sleep 2

# 5. Verify
btmgmt info
# Expected: Index list with 1 item, hci0: Primary controller

sudo bluetoothctl power on
sudo bluetoothctl scan on
# Expected: Changing power on succeeded, device appears in scan
```

#### Option B: Permanent Fix (via systemd)

```bash
# 1. Open service for editing
sudo systemctl edit orangepi3b-sprd-bluetooth

# 2. Override ExecStart (remove -n):
[Service]
ExecStart=
ExecStart=/usr/bin/hciattach_opi -s 1500000 /dev/ttyBT0 sprd

# 3. Apply and restart
sudo systemctl daemon-reload
sudo systemctl restart orangepi3b-sprd-bluetooth
sudo systemctl restart bluetooth

# 4. Verify
btmgmt info
sudo bluetoothctl power on
sudo bluetoothctl scan on
```

> 💡 **Why `-n` breaks things**: The `no-detach` flag keeps `hciattach` running in the background and holding the UART. BlueZ can't register the controller because the device is already busy.

---

## 🔧 If It Doesn't Work: Diagnostics

```bash
# Check if kernel sees the chip
dmesg | grep -iE "ttyBT|sprd|uwe5622"

# Check if drivers are loaded
lsmod | grep -iE "sprd|bt"

# Check for blocks
rfkill list
# If Soft blocked: yes → sudo rfkill unblock bluetooth

# Check if bluetoothd is running with required flags
ps aux | grep bluetoothd
# For passive scanning (required for HA): needs --experimental

# Check device permissions
ls -l /dev/hci0 /dev/ttyBT0
# Should be: root:bluetooth, permissions 660
```

---

## 🐳 For Docker: Passing Through to Container

If you're running Home Assistant or another app in Docker:

```yaml
services:
  home-assistant:
    volumes:
      - /run/dbus:/run/dbus:ro # ← Required for Bluetooth access
    # Don't run bluetoothctl inside the container!
    # Manage pairing on the host, the app will pick it up via D-Bus
```

> ⚠️ **Don't use `network_mode: host` just for Bluetooth** - it breaks network isolation. Mounting `/run/dbus` is enough.

---

## Links

- 🍊 [Orange Pi 3B Wiki](https://wiki.orangepi.org/)
- 📘 [BlueZ Documentation](https://www.bluez.org/)
- 🔧 [hciattach Source](https://git.kernel.org/pub/scm/bluetooth/bluez.git/tree/tools/hciattach.c)
- 🐙 [Meshtastic Docs](https://meshtastic.org/docs/configuration/)
