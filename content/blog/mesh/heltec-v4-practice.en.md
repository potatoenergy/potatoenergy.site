---
author: ["Potato Energy Team", "ponfertato"]
categories: ["hardware", "mesh", "lora", "tutorial"]
date: "2026-04-27T10:30:00+03:00"
description: "Heltec V4: unboxing, flashing, MQTT setup for ONEmesh, first messages between nodes. Part 2: Practice."
draft: false
series: ["Mesh Networks"]
slug: "practice"
tags:
  ["Heltec", "esp32", "meshtastic", "firmware", "mqtt", "onemesh", "tutorial"]
title: "Heltec V4: Practice. Flashing, Setup, First Messages. Part 2"
---

> 📌 **This is Part 2 of the series**. [Part 1: Theory]({{< ref "/blog/mesh/heltec-v4-mesh-intro.en.md" >}}) explains why this matters.

---

## 📦 Unboxing and Preparation

### Heltec V4 Contents

| Component       | Purpose                                 |
| --------------- | --------------------------------------- |
| Board ~60×30 mm | ESP32-S3R2 + SX1262, OLED 128×64, USB-C |
| Antenna (IPEX)  | Connect to `ANT` port - **mandatory!**  |
| USB-C cable     | Power + flashing + debugging            |
| Pins (optional) | For external sensors/antennas           |

> ⚠️ **Important**: without antenna connected, the radio module may be damaged. Always attach antenna before powering on.

### Heltec V4 Specifications

| Component            | Description                                                |
| -------------------- | ---------------------------------------------------------- |
| **MCU**              | ESP32-S3R2 (WiFi + Bluetooth)                              |
| **LoRa Transceiver** | Semtech SX1262                                             |
| **Frequencies**      | 863–870 MHz (EU), 902–928 MHz (US)                         |
| **Display**          | 0.96" OLED 128×64                                          |
| **Power**            | Up to +28±1 dBm (High Power option)                        |
| **Power Supply**     | USB-C + optimized LiPo management                          |
| **Connectors**       | USB-C, U.FL/IPEX for LoRa, 1.25-8Pin GNSS, 1.25-2Pin Solar |
| **Form Factor**      | Pin-compatible with V3/V3.1                                |

---

## ⚡ Flashing: Web Flasher

Official flasher: [https://flasher.meshtastic.org](https://flasher.meshtastic.org)

### Requirements

| Requirement                                      | Why                                        |
| ------------------------------------------------ | ------------------------------------------ |
| **Chromium-based browser** (Chrome, Edge, Brave) | Web Serial API works only there            |
| **Serial port access**                           | On Linux - user must be in `dialout` group |
| **Battery disconnected**                         | During flashing - USB power only           |

### 🐧 Linux: Serial Port Permissions

If you see:

```
[09:52:42.841] Serial: serial_io_handler.cc:157 Failed to open serial port: FILE_ERROR_ACCESS_DENIED
```

**Fix**:

```bash
# Add user to dialout group
sudo usermod -a -G dialout $USER

# Apply changes (re-login or)
newgrp dialout

# Verify access
ls -l /dev/ttyUSB*  # or /dev/ttyACM*
```

### 🔧 Flashing Process

1. **Disconnect** battery and USB from the board
2. Open [web flasher](https://flasher.meshtastic.org), select:
   - Device: `Heltec V4`
   - Firmware: `2.7.21` (or latest stable/beta)
   - Variant: `Full erase and install`
3. Click **Erase Flash and Install**
4. **Hold the `PRG` button** on the board
5. **While holding `PRG`**, plug in the USB cable
6. Observe: brief red flash → `USB JTAG` appears in port dropdown
7. Release `PRG`, select `USB JTAG`, click **Connect**
8. Wait for flashing to complete (terminal shows progress)
9. After `Leaving...` message, **press the `RST` button** (reset)

> 💡 If flasher doesn't detect the port - try another cable (not all support data), another USB port, or restart the browser.

---

## 🔌 First Connection: Three Ways

After flashing, the board reboots and starts Meshtastic.

### Connection Options

| Interface     | How to Connect                                                 | When to Use                                      |
| ------------- | -------------------------------------------------------------- | ------------------------------------------------ |
| **Bluetooth** | Meshtastic app (Android/iOS) → scan devices                    | Portable node, one active connection             |
| **Wi-Fi**     | Node creates AP → connect to your network → IP shown on screen | Stationary node, multiple devices simultaneously |
| **USB/COM**   | Connect via terminal (PuTTY, screen)                           | Home node, debugging, direct access              |

> ⚠️ **Bluetooth**: only one active connection.  
> **Wi-Fi**: multiple devices can connect simultaneously (convenient for stationary nodes).  
> **USB**: reliable for setup and debugging, no wireless connection required.

### Setup via App (Recommended)

1. Install [Meshtastic](https://meshtastic.org/docs/software/android/installation/) (Android) or [iOS version](https://apps.apple.com/app/meshtastic/id1686803353)
2. Launch, grant location and Bluetooth permissions
3. Tap `+` → select your node (by name `Meshtastic_XXXX`)
4. Confirm pairing

### Setup via Wi-Fi

1. Node screen displays IP address (usually `192.168.1.x`)
2. Connect to node's Wi-Fi network (password shown on screen)
3. Open `http://<node_ip>` in browser
4. Configure via web interface

> 💡 **IP Address**: node gets address from your router (not `192.168.4.1`!). Exact address displayed on OLED screen.

---

## 👤 User Configuration

### Core Parameters

| Parameter      | Value                     | Note                                       |
| -------------- | ------------------------- | ------------------------------------------ |
| **Long Name**  | `[PE] ponfertato`         | Full name (up to 20 characters)            |
| **Short Name** | `ponf`                    | Short name (**maximum 4 characters**)      |
| **Role**       | `CLIENT` or `CLIENT_MUTE` | For cities with >200 nodes - `CLIENT_MUTE` |

> 💡 **ShortName**: strictly 4 characters! This is the identifier in messages.

---

## 🌐 MQTT Setup for ONEmesh (RU)

[ONEmesh](https://map.onemesh.ru) - map of Meshtastic devices in Russia. Connecting to their MQTT server lets your node appear on the map and exchange data with other participants.

> 🔐 **MQTT transmits**: messages, device metrics, location (if enabled) to the map and Telegram chats according to region/city settings.

### Core Parameters

| Parameter               | Value                            | Note                                                         |
| ----------------------- | -------------------------------- | ------------------------------------------------------------ |
| **MQTT Address**        | `mqtt.onemesh.ru`                | Community server                                             |
| **Username**            | `onemesh`                        | Default; `onemeshz` / `onemeshd` - for downlink              |
| **Password**            | `onecat`                         | Shared for all                                               |
| **Encryption enabled**  | `Yes`                            | If channels have PSK                                         |
| **JSON enabled**        | `No`                             | Not needed for map                                           |
| **TLS enabled**         | `Yes`                            | If not working - try `No`                                    |
| **Root topic**          | `msh/RU/XXX`                     | `XXX` = city code ([list](https://map.onemesh.ru/mqtt.html)) |
| **Proxy to client**     | `Yes` (Bluetooth) / `No` (Wi-Fi) | Depends on connection method                                 |
| **Map reports**         | `Yes`                            | Send position to map                                         |
| **I agree**             | `Yes`                            | Accept terms                                                 |
| **Precision**           | `729 m`                          | Coordinate precision on map                                  |
| **Map report interval** | `3600 s` (1 hour)                | Minimum interval                                             |

> 💡 **Tip**: if "Save" button is inactive - fill settings in stages: first basics (address, login, password), then map, then extra options.

### Username Modes

| Username   | Downlink    | When to Use                                                                               |
| ---------- | ----------- | ----------------------------------------------------------------------------------------- |
| `onemesh`  | ❌ Disabled | Most users - safe mode                                                                    |
| `onemeshz` | ✅ Zero-hop | If you need internet data but won't rebroadcast over radio                                |
| `onemeshd` | ✅ Full     | Only for integrations, connecting network segments; **not recommended** for regular nodes |

> ⚠️ Using `onemeshd` with Downlink enabled adds load to the radio network. Use consciously.

---

## 📡 LoRa and Channel Settings

### LoRa Section

| Parameter       | Value    | Why                                             |
| --------------- | -------- | ----------------------------------------------- |
| **Region**      | `Russia` | Auto-sets 433 MHz frequency and MQTT root topic |
| **Hop limit**   | `5`      | Max message hops; increase for large networks   |
| **Ignore MQTT** | `No`     | Accept packets that came via internet           |
| **OK to MQTT**  | `Yes`    | Allow neighbors to forward your data to map     |

> 🔁 If you change region - verify `Root topic` in MQTT updated automatically.

### Channels

Configure the **Primary channel**:

| Parameter            | Value   | Note                                                            |
| -------------------- | ------- | --------------------------------------------------------------- |
| **PSK**              | `AQ==`  | Base encryption key; required if MQTT encryption is on          |
| **Uplink enabled**   | `Yes`   | Send data from this channel to MQTT                             |
| **Downlink enabled** | `No`    | Receive data from MQTT (enable only with `onemeshz`/`onemeshd`) |
| **Position enabled** | `Yes`   | Transmit coordinates via this channel                           |
| **Precise location** | `No`    | Hide exact coordinates from other nodes                         |
| **Precision**        | `182 m` | Position precision in channel                                   |

> ✅ Setting one channel with `PSK`, `Position enabled`, and `Precision` is enough - even without MQTT, a neighboring node can forward your data.

---

## 📍 Position Settings

### GPS Device

| Parameter          | Value               | Recommendation                |
| ------------------ | ------------------- | ----------------------------- |
| **Fixed position** | Enabled (if no GPS) | Enter coordinates manually    |
| **Latitude**       | `57.74213199999996` | Latitude (no trailing zeros)  |
| **Longitude**      | `40.9780224`        | Longitude (no trailing zeros) |
| **Altitude**       | `265`               | Altitude in meters            |

> 💡 **Coordinates**: enter without trailing zeros (e.g., `57.742`, not `57.74200`).

### Position Packet

| Parameter                       | Value                   | Why                                                |
| ------------------------------- | ----------------------- | -------------------------------------------------- |
| **Position broadcast interval** | `3600–43200 s` (1–12 h) | In dense cities - 6–12 h to reduce load            |
| **Smart position**              | `Yes`                   | Sends more often when moving, less when stationary |
| **Smart interval**              | `30 seconds`            | Minimum interval when position changes rapidly     |

> 🔐 For privacy: reduce `Precision` in channel and `Map precision`, or set fixed coordinates away from real location.

---

## ⚙️ Additional Modules

### Environment Module

| Parameter              | Value           | Why                                          |
| ---------------------- | --------------- | -------------------------------------------- |
| **Enabled**            | `Yes`           | Send neighbor data to map                    |
| **Update interval**    | `14400 s` (4 h) | Minimum interval                             |
| **Transmit over LoRa** | `No`            | Don't load radio network; data goes via MQTT |

> 🗺️ Map will show layers "Who heard this device" and "Who this device heard".

### Device Settings

| Parameter              | Value                | When to Use                                               |
| ---------------------- | -------------------- | --------------------------------------------------------- |
| **Role**               | `CLIENT_MUTE`        | In cities with >200 nodes - safe mode without rebroadcast |
|                        | `CLIENT`             | If network is small or you have good antenna/location     |
| **Rebroadcast mode**   | `CORE_PORTNUMS_ONLY` | For large networks; `ALL` - for small ones                |
| **Broadcast interval** | `3 hours`            | Interval for node information transmission                |

### Timezone

| Parameter              | Value                               |
| ---------------------- | ----------------------------------- |
| **Timezone**           | `GMT+3` (Moscow)                    |
| **Use phone timezone** | Enabled (if configuring from phone) |

---

## 🔧 Optimal Flashing Process

### For Heltec V4 (Bootloader Workaround)

Due to a Heltec V4 bootloader quirk, the standard order doesn't always work. Optimal process:

1. **Disconnect** battery and USB from the board
2. Open [web flasher](https://flasher.meshtastic.org)
3. Select device: `Heltec V4`, firmware: `2.7.21`
4. Click **Erase Flash and Install**
5. **Hold the `PRG` button** on the board
6. **While holding `PRG`**, plug in the USB cable
7. Observe: brief red flash → `USB JTAG` appears in port dropdown
8. Release `PRG`, select `USB JTAG`, click **Connect**
9. Wait for flashing to complete
10. After `Leaving...` message, **press the `RST` button** (reset)

> 💡 **Note**: this process is described in [meshtastic/firmware#8543](https://github.com/meshtastic/firmware/issues/8543)

---

## 📤 First Messages Between Two Nodes

### Preparation

1. Flash **both** boards per instructions above
2. On each, configure:
   - Same region (`Russia`)
   - Same channel with identical `PSK` (`AQ==`)
   - `Position enabled: Yes` (for testing)
3. Place nodes 10–50 m apart (for initial test)

### Test

1. On first node in app: `Messages` → `+` → type text → `Send`
2. On second node: incoming message should appear
3. Check `Map` tab - if MQTT is enabled, both nodes should appear on map within an hour

### Troubleshooting

| Symptom               | Check                                                       | Fix                                                   |
| --------------------- | ----------------------------------------------------------- | ----------------------------------------------------- |
| No messages           | Verify channels have same `PSK`                             | Copy key exactly, no spaces                           |
| Node not on map       | Check `MQTT enabled`, `Map reports`, internet on phone/node | Reboot node (`RST`), wait 1–2 report cycles           |
| MQTT connection error | Verify `TLS enabled`, root topic                            | Try disabling TLS; ensure `Root topic` = `msh/RU/XXX` |
| No radio connection   | Check antenna, region, `Hop limit`                          | Attach antenna, set `Region: Russia`, `Hop limit: 5`  |

---

## 🗓 What's Next

1. ✅ **Part 2: Practice** - unboxing, flashing, first messages (this article)
2. 🔜 **Part 3: Field Tests** - urban range tests, antenna tuning, power consumption
3. 🔜 **Part 4: Integrations** - Home Assistant, Telegram bridge, sensors
4. 🔜 **Part 5: Advanced** - Reticulum, custom firmware, optimization

---

## 🆘 Help and Community

- 🗺️ [ONEmesh Map](https://map.onemesh.ru) - network visualization
- 💬 [Telegram Group](https://t.me/onemesh_ru) - questions, help, city coordination
- 📘 [Official Meshtastic Docs](https://meshtastic.org/docs/)
- 🐙 [Firmware Repository](https://github.com/meshtastic/firmware)
- 🔧 [Heltec V4 Specs](https://meshtastic.org/docs/hardware/devices/Heltec-automation/lora32/?Heltec=v4)
