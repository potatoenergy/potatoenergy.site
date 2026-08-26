---
author: ["Potato Energy Team", "ponfertato"]
categories: ["android", "adb", "linux", "guide"]
date: "2026-08-26T17:00:00+03:00"
description: "Scrcpy: wireless mirroring and control of Android from PC. Troubleshooting ADB connection methods and common errors."
draft: false
series: ["Android Tips"]
slug: "scrcpy-wireless"
tags: ["android", "scrcpy", "adb", "wireless", "linux"]
title: "Scrcpy: Wireless Mirroring and Control of Android from PC"
---

## 🎯 What is Scrcpy and Why Use It

[Scrcpy](https://github.com/Genymobile/scrcpy) (Screen Copy) is the gold standard for displaying and controlling Android devices from a computer.
**Benefits**:

- ✅ No root required
- ✅ Low latency (35~70 ms)
- ✅ High quality (up to 1920×1080 or higher)
- ✅ Works over USB and **Wi-Fi**

---

## ⚠️ The Main Trap: "Connection refused"

Many users try to start wireless mode immediately with:

```bash
scrcpy --tcpip=192.168.x.x
```

And get the error: `failed to connect to '192.168.x.x:5555': Connection refused`.

**Reason**: By default, the Android debugging daemon (`adbd`) listens only on the USB interface. Port `5555` is closed until you explicitly switch the device to TCP/IP mode.

---

## 🔧 3 Connection Methods (From Easiest to Advanced)

### Method 1: The One-Command Magic (Recommended)

The simplest method, requiring a USB cable only for initialization.

1. Connect your phone to the PC via **USB**.
2. Ensure USB debugging is enabled (and you tapped "Allow" on the phone screen on first connection).
3. Run the command **without specifying the IP address**:
   ```bash
   scrcpy --tcpip
   ```
4. **What happens**: Scrcpy automatically switches the device to TCP/IP mode, detects its Wi-Fi IP address, and connects.
5. Once connected, **unplug the USB cable**. The mirroring will continue over Wi-Fi.

---

### Method 2: Classic Manual (via ADB)

Useful for understanding the mechanics or if Method 1 fails.

1. Connect the device via **USB**.
2. Switch `adbd` to listen on the network:
   ```bash
   adb tcpip 5555
   ```
   _(Expected output: `restarting in TCP mode port: 5555`)_
3. **Unplug the USB cable**.
4. Connect to the device using its IP address (find it in phone Wi-Fi settings or via `adb shell ip route`):
   ```bash
   adb connect 192.168.x.x:5555
   ```
5. Start the mirroring:
   ```bash
   scrcpy
   ```

---

### Method 3: Truly Wireless (Android 11+)

If you don't have a USB cable, but both phone and PC are on the same Wi-Fi network.

1. On phone: **Settings** → **Developer options** → **Wireless debugging** (enable).
2. Tap **"Pair device with pairing code"**. The screen will show an IP address, port, and a 6-digit code.
3. On PC, initiate pairing (use the IP and port from step 2):
   ```bash
   adb pair 192.168.x.x:PORT
   # Enter the 6-digit code from the phone screen
   ```
4. Go back to the "Wireless debugging" menu on the phone. It will show a **different** IP and port for actual connection (not for pairing).
5. Connect:
   ```bash
   adb connect 192.168.y.y:PORT
   ```
6. Run `scrcpy`.

---

## 🚨 Troubleshooting Common Errors (From Experience)

### Error 1: `failed to authenticate to 192.168.x.x:5555`

**Symptom**: Command executes, but connection is dropped immediately.
**Reason**: A system prompt is waiting on the phone screen: _"Allow USB debugging from this computer?"_ with an RSA key fingerprint. Until you tap **"Allow"** (and ideally check "Always allow"), `adbd` will reject the authentication.
**Solution**: Unlock the phone screen and confirm the prompt.

### Error 2: `error: protocol fault (couldn't read status message): Success` during `adb pair`

**Symptom**: Error when attempting to pair.
**Reason**: Mode conflict. The `adb pair` command works **only** with the "Wireless debugging" feature (Android 11+). If you previously ran `adb tcpip 5555`, the device is already in classic mode and will not accept the pairing protocol.
**Solution**: Use either Method 1/2 (classic `tcpip`) OR Method 3 (`pair`). Do not mix them.

---

## 💡 Pro Tips for Scrcpy

For maximum comfort, use these flags when launching:

| Flag                | Description                                                 | Example                    |
| ------------------- | ----------------------------------------------------------- | -------------------------- |
| `--max-size 1024`   | Limits resolution (saves Wi-Fi bandwidth)                   | `scrcpy --max-size 1024`   |
| `--bit-rate 2M`     | Limits bitrate (default is 8M)                              | `scrcpy --bit-rate 2M`     |
| `--turn-screen-off` | Turns off the phone screen during mirroring (saves battery) | `scrcpy --turn-screen-off` |
| `--stay-awake`      | Prevents the phone from sleeping while running              | `scrcpy --stay-awake`      |
| `--no-audio`        | Disables audio forwarding (if you only need the screen)     | `scrcpy --no-audio`        |

**Combo for weak Wi-Fi**:

```bash
scrcpy --max-size 800 --bit-rate 1M --turn-screen-off --stay-awake
```
