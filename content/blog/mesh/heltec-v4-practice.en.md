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

| Parameter             | Value                     | Note                                                                                          |
| --------------------- | ------------------------- | --------------------------------------------------------------------------------------------- |
| **Long Name**         | `[PE] ponfertato`         | Full node name, visible to all (up to 20 chars, Cyrillic allowed)                             |
| **Short Name**        | `ponf`                    | Short identifier in messages (**strictly 4 chars**, Latin only)                               |
| **Role**              | `CLIENT` or `CLIENT_MUTE` | For cities with >200 nodes - `CLIENT_MUTE` (safe mode without rebroadcasting others' packets) |
| **Is Licensed (HAM)** | `No`                      | If `Yes` → disable MQTT encryption (regulatory requirement)                                   |

> 💡 **ShortName**: exactly 4 characters! This appears in message headers. Examples: `ponf`, `KST1`, `MOW2`.

### Additional Flags (Optional)

| Parameter                    | Value | Why                                                |
| ---------------------------- | ----- | -------------------------------------------------- |
| **Show on map**              | `Yes` | Allow display on public maps (controlled via MQTT) |
| **Ignore location requests** | `No`  | Respond to coordinate requests from other nodes    |

### Advanced User Settings

| Parameter        | Value              | Note                                         |
| ---------------- | ------------------ | -------------------------------------------- |
| **Public Key**   | _(auto-generated)_ | Don't change manually; used for encryption   |
| **Private Key**  | _(keep secret)_    | Export and save securely on first setup      |
| **Admin Key**    | _(empty)_          | Fill only for remote node management         |
| **Managed Mode** | `No`               | Don't enable without configured Remote Admin |

---

## 🌐 MQTT Setup for ONEmesh (RU)

[ONEmesh](https://map.onemesh.ru) - map of Meshtastic devices in Russia. Connecting to their MQTT server lets your node appear on the map and exchange data with other participants.

> 🔐 **What's transmitted via MQTT**: text messages, device metrics (battery, signal), location (if enabled). Data goes to the map and regional Telegram chats per city settings.

### Core Parameters

| Parameter               | Value                            | Note                                                                              |
| ----------------------- | -------------------------------- | --------------------------------------------------------------------------------- |
| **MQTT Address**        | `mqtt.onemesh.ru`                | Community server (domain, not IP)                                                 |
| **Port**                | `8883` (TLS) / `1883` (no TLS)   | Port auto-selected when TLS enabled                                               |
| **Username**            | `onemesh`                        | Default; `onemeshz` / `onemeshd` for downlink                                     |
| **Password**            | `onecat`                         | Shared for all project participants                                               |
| **Encryption enabled**  | `Yes`                            | Encrypt MQTT traffic; requires `PSK: AQ==` in channel                             |
| **JSON enabled**        | `No`                             | Not needed for map, adds overhead                                                 |
| **TLS enabled**         | `Yes`                            | Encrypt connection; if errors → try `No` + port `1883`                            |
| **Root topic**          | `msh/RU/KST`                     | `KST` = Kostroma city code ([code list](https://map.onemesh.ru/city-topics.html)) |
| **Proxy to client**     | `Yes` (Bluetooth) / `No` (Wi-Fi) | Not needed with direct Wi-Fi                                                      |
| **Map reports**         | `Yes`                            | Allow sending reports for map display                                             |
| **I agree**             | `Yes`                            | Confirm geodata transmission rules                                                |
| **Precision**           | `729 m`                          | Coordinate precision on public map (~15 bits)                                     |
| **Map report interval** | `3600 s` (1 hour)                | Minimum report sending interval                                                   |

> 💡 **Tip**: if "Save" button is inactive - fill settings in stages: basics first (address, login, password), then map, then extra options.

### Username Modes

| Username   | Downlink    | When to Use                                                                                                   |
| ---------- | ----------- | ------------------------------------------------------------------------------------------------------------- |
| `onemesh`  | ❌ Disabled | Most users - safe mode, uplink only                                                                           |
| `onemeshz` | ✅ Zero-hop | If you need internet data but won't rebroadcast over radio (zero-hop policy)                                  |
| `onemeshd` | ✅ Full     | Only for integrations, connecting network segments; **not recommended** for regular nodes - adds airtime load |

> ⚠️ Using `onemeshd` with Downlink enabled can overload local network with internet packets. Use consciously and only with full understanding.

### Advanced MQTT Parameters

| Parameter                     | Value          | Note                                                             |
| ----------------------------- | -------------- | ---------------------------------------------------------------- |
| **Connection Retry Interval** | `30` sec       | How often to retry reconnect on drop                             |
| **Keepalive Interval**        | `60` sec       | Connection liveness check interval                               |
| **QoS Level**                 | `0`            | Quality of Service: 0 = fire-and-forget (optimal for Meshtastic) |
| **Retain Messages**           | `No`           | Don't retain last messages on broker                             |
| **Topic Filter**              | `msh/RU/KST/#` | Subscribe to subtopics (for integrations)                        |

### Common MQTT Issues

| Symptom                 | Possible Cause                            | Solution                                         |
| ----------------------- | ----------------------------------------- | ------------------------------------------------ |
| `Connection refused`    | Wrong login/password or port              | Check `onemesh`/`onecat`, port `8883` with TLS   |
| `TLS handshake failed`  | Certificate or time sync issue            | Check NTP sync, temporarily disable TLS for test |
| `No messages on map`    | Wrong `Root topic` or `PSK`               | Verify: `msh/RU/KST`, channel with `PSK: AQ==`   |
| `MQTT disconnected`     | Unstable internet on node/phone           | Check connection, increase `Map report interval` |
| `Message not appearing` | Downlink disabled on server for `onemesh` | Use `onemeshz` if downlink needed                |

---

## 📡 LoRa and Channel Settings

### LoRa Section

| Parameter           | Value      | Why                                                              |
| ------------------- | ---------- | ---------------------------------------------------------------- |
| **Region**          | `Russia`   | Auto-sets 433 MHz, power ≤20 dBm, and MQTT root topic            |
| **Modem Preset**    | `LongFast` | Range/speed balance for urban conditions                         |
| **Frequency Slot**  | `2`        | Public interval for `RU868` (avoid conflicts)                    |
| **Transmit Power**  | `20` dBm   | Max allowed for amateur use in RF                                |
| **Boosted RX Gain** | `On`       | Improves weak signal reception (critical for urban)              |
| **Hop Limit**       | `5`        | Max hops; enough for city, doesn't overload airtime              |
| **Ignore MQTT**     | `No`       | Accept packets that came via internet (from MQTT neighbors)      |
| **OK to MQTT**      | `Yes` ⚠️   | **Critical**: allows neighbors to relay your data to OneMesh map |

### Advanced LoRa Parameters

| Parameter                  | Value     | Note                                                      |
| -------------------------- | --------- | --------------------------------------------------------- |
| **Bandwidth**              | `250` kHz | Channel bandwidth; don't change when using presets        |
| **Spreading Factor**       | `11`      | Spreading factor; higher = farther but slower             |
| **Coding Rate**            | `5` (4/5) | Error correction; balance reliability/speed               |
| **Frequency Offset**       | `0.0` Hz  | Crystal frequency correction; change only for calibration |
| **Override Duty Cycle**    | `No`      | Don't override airtime restrictions                       |
| **SX126x RX Boosted Gain** | `On`      | Hardware RX gain boost on SX1262                          |

### Channels - Detailed

Configure the **Primary channel** (index 0):

| Parameter            | Value      | Note                                                                           |
| -------------------- | ---------- | ------------------------------------------------------------------------------ |
| **Role**             | `Primary`  | Only one channel can be primary; device telemetry goes through it              |
| **Name**             | `LongFast` | **Exactly this**: public network filters packets by this name                  |
| **PSK**              | `AQ==`     | Base encryption key for OneMesh (128-bit); required if MQTT encryption enabled |
| **Uplink enabled**   | `Yes`      | Send data from this channel to MQTT server                                     |
| **Downlink enabled** | `No`       | Don't receive from MQTT (unless using `onemeshz`/`onemeshd`)                   |
| **Position enabled** | `Yes`      | Transmit coordinates via this channel                                          |
| **Precise location** | `No`       | Hide exact coordinates from other nodes in channel                             |
| **Precision**        | `182 m`    | Position precision transmitted in channel (~17 bits)                           |
| **Muted**            | `No`       | Don't mute channel (otherwise messages won't be visible)                       |

#### Secondary Channel (Optional, Index 1)

| Parameter             | Value                 | Note                                          |
| --------------------- | --------------------- | --------------------------------------------- |
| **Role**              | `Secondary`           | Additional channel for private groups or bots |
| **Name**              | `[PE] Backup`         | Any name <12 bytes (Latin)                    |
| **PSK**               | _(generate your own)_ | Private key for your group                    |
| **Uplink / Downlink** | `No`                  | Don't send to public MQTT                     |
| **Position**          | `No`                  | Don't transmit coordinates in private channel |
| **Precision**         | `45 m`                | Maximum precision for private use             |

> ✅ **Important**: at least one channel must have `PSK: AQ==` **or** be unencrypted for OneMesh server to accept packets.

> 💡 Secondary channels are created via `+` in the app. They don't interfere with main traffic and are useful for private communication.

### Channel Management: Tips

- **Channel order**: Primary must be first (index 0) - system telemetry goes through it.
- **Channel names**: use Latin, no spaces or special characters.
- **PSK lengths**: supported: 0 (no encryption), 8, 128, 256 bits. OneMesh requires `AQ==` (8-bit).
- **Export/import**: channel settings can be exported to JSON for quick deployment on other nodes.

---

## 📍 Position Settings

### Fixed Coordinates (for Stationary Nodes)

| Parameter          | Value    | Recommendation                                                               |
| ------------------ | -------- | ---------------------------------------------------------------------------- |
| **Fixed position** | `On`     | For nodes without GPS module                                                 |
| **Latitude**       | `57.742` | Kostroma latitude, **no trailing zeros** (enter as `57.742`, not `57.74200`) |
| **Longitude**      | `40.978` | Kostroma longitude, same as above                                            |
| **Altitude**       | `100`    | Altitude above sea level (approx., in meters)                                |

> 💡 Coordinates stored as `integer × 1e7`. In app, enter with 6 decimal places.

### Position Packet - Intervals and Precision

| Parameter                       | Value                  | Why                                                                    |
| ------------------------------- | ---------------------- | ---------------------------------------------------------------------- |
| **Position broadcast interval** | `7200 s` (2 hours)     | Balance between map freshness and airtime load                         |
| **Smart position**              | `No`                   | Fixed interval more reliable for stationary node                       |
| **Smart position min distance** | `100 m`                | Min distance to trigger smart broadcast                                |
| **Smart position min interval** | `300 s`                | Min interval between smart broadcasts                                  |
| **Position flags**              | `ALTITUDE + TIMESTAMP` | Transmit only altitude and timestamp (less traffic)                    |
| **Provide location to network** | `Yes`                  | Allow coordinate transmission to network (controlled via `OK to MQTT`) |

### Advanced Position Settings

| Parameter                            | Value     | Note                                    |
| ------------------------------------ | --------- | --------------------------------------- |
| **GPS Enabled**                      | `No`      | Disable if no module; saves power       |
| **GPS Update Interval**              | `60` sec  | How often to poll GPS (if enabled)      |
| **GPS Attempt Time**                 | `0`       | Don't limit satellite search time       |
| **RX/TX/Enable GPIO**                | `0`       | Pins for external GPS module (defaults) |
| **Broadcast Smart Minimum Distance** | `100` m   | Min movement to trigger smart broadcast |
| **Broadcast Smart Minimum Interval** | `300` sec | Min time between smart broadcasts       |

> 🔐 For privacy: reduce `Precision` in channel (`182 m`) and `Map precision` (`729 m`), or set fixed coordinates away from real location.

---

## ⚙️ Additional Modules

### Neighbor Info

| Parameter              | Value               | Note                                                    |
| ---------------------- | ------------------- | ------------------------------------------------------- |
| **Enabled**            | `Yes`               | For "Neighbors" layer on OneMesh map                    |
| **Update interval**    | `14400 s` (4 hours) | Minimum interval, don't reduce - extra airtime load     |
| **Transmit over LoRa** | `No`                | Don't transmit neighbor data over radio (only via MQTT) |

> 🗺️ Map will show layers "Who heard this device" and "Who this device heard" - useful for coverage analysis.

### Device - Role and Rebroadcast

| Parameter                        | Value                | When to Use                                                                  |
| -------------------------------- | -------------------- | ---------------------------------------------------------------------------- |
| **Role**                         | `CLIENT_MUTE`        | In cities with >200 nodes - safe mode without rebroadcasting others' packets |
|                                  | `CLIENT`             | If network is small or you have good antenna/location - can help network     |
| **Rebroadcast mode**             | `CORE_PORTNUMS_ONLY` | Rebroadcast only basic packet types (position, text) - optimal for city      |
|                                  | `ALL`                | Rebroadcast everything - only for tests or very small networks               |
| **Node info broadcast interval** | `43200 s` (12 hours) | How often to broadcast info about yourself; 12h is enough for monitoring     |
| **Double tap as button**         | `No`                 | Prevent accidental triggers                                                  |
| **Disable triple click**         | `Yes`                | Prevent false beacons                                                        |
| **LED heartbeat disabled**       | `No`                 | Visual operation indicator (LED blinking)                                    |

### Advanced Device Settings

| Parameter          | Value         | Note                                                  |
| ------------------ | ------------- | ----------------------------------------------------- |
| **Button GPIO**    | `0`           | Button pin (default)                                  |
| **Buzzer GPIO**    | `0`           | Buzzer pin (default)                                  |
| **Buzzer Mode**    | `ALL_ENABLED` | Buzzer mode: `DISABLED`, `ALERTS_ONLY`, `ALL_ENABLED` |
| **TZDEF**          | `GMT-3`       | Timezone for local time                               |
| **Is Managed**     | `No`          | Managed mode (only for admin nodes)                   |
| **Serial Enabled** | `No`          | Serial console (for debugging)                        |
| **Debug Log API**  | `No`          | Output debug logs (not for production)                |

### Timezone

| Parameter              | Value            | Note                                                            |
| ---------------------- | ---------------- | --------------------------------------------------------------- |
| **Timezone**           | `GMT+3` (Moscow) | For correct time display in logs and timestamps                 |
| **Use phone timezone** | `Button`         | Press to populate the timezone from the phone's system settings |

### Power - for Stability

| Parameter                     | Value   | Note                                           |
| ----------------------------- | ------- | ---------------------------------------------- |
| **Power Saving Mode**         | `No`    | Stationary node with USB power                 |
| **On Battery Shutdown After** | `0` sec | Don't shut down on power loss (if UPS present) |
| **ADC Multiplier Override**   | `1.0`   | Battery voltage calibration                    |
| **Wait Bluetooth Secs**       | `0`     | With direct WiFi, don't wait for BLE           |
| **Min Wake Secs**             | `30`    | Min active time after receiving packet         |
| **INA219 Address**            | `0`     | Power monitor address (auto-detect)            |

### Display - OLED Optimization

| Parameter                 | Value       | Note                                                       |
| ------------------------- | ----------- | ---------------------------------------------------------- |
| **Screen Timeout**        | `60` sec    | OLED timeout for power saving                              |
| **GPS Format**            | `UNUSED`    | Coordinate format on screen (not used with fixed position) |
| **Auto Screen Carousel**  | `0`         | Disable auto window rotation                               |
| **Compass North Top**     | `Yes`       | Fix north to top of compass                                |
| **Flip Screen**           | `No`        | Don't flip screen (per body orientation)                   |
| **Units**                 | `METRIC`    | Metric system (°C, m, km/h)                                |
| **OLED Type**             | `OLED_AUTO` | Auto-detect display controller                             |
| **Display Mode**          | `DEFAULT`   | Standard display mode                                      |
| **Heading Bold**          | `Yes`       | Bold heading for better readability                        |
| **Wake on Tap or Motion** | `No`        | Disable wake on motion (no accelerometer on V4)            |

### Bluetooth - When WiFi Off

| Parameter        | Value                  | Note                                                |
| ---------------- | ---------------------- | --------------------------------------------------- |
| **Enabled**      | `No`                   | With WiFi enabled, Bluetooth auto-disables on ESP32 |
| **Pairing Mode** | `FIXED_PIN`            | If enabling - use fixed PIN                         |
| **Fixed PIN**    | `123456` → **change!** | Must change to random 6-digit code                  |

### Network - WiFi for Stationary Node

| Parameter                      | Value          | Note                                                 |
| ------------------------------ | -------------- | ---------------------------------------------------- |
| **WiFi Enabled**               | `Yes`          | Direct connection to router                          |
| **SSID / PSK**                 | `<your_data>`  | 2.4 GHz networks only                                |
| **Enable Local UDP Broadcast** | `No`           | Not required for OneMesh                             |
| **NTP Server**                 | `pool.ntp.org` | More reliable than default `meshtastic.pool.ntp.org` |
| **Address Mode**               | `DHCP`         | Auto IP assignment                                   |
| **IPv6 Enabled**               | `No`           | Not used in current config                           |

> ⚠️ With WiFi enabled, Bluetooth auto-disables on ESP32 architecture.

---

## 🧩 Module Settings (Continued)

### Telemetry

| Parameter                       | Value              | Note                              |
| ------------------------------- | ------------------ | --------------------------------- |
| **Send Device Telemetry**       | `Yes`              | Enable for node status monitoring |
| **Device Metrics Interval**     | `900` sec (15 min) | How often to send device metrics  |
| **Power Metrics Enabled**       | `Yes`              | Critical for power monitoring     |
| **Power Metrics Interval**      | `300` sec (5 min)  | How often to send power data      |
| **Environment Metrics Enabled** | `No`               | If no BME280/BME680 sensors       |
| **Air Quality Enabled**         | `No`               | Only for BME680                   |
| **Display Metrics on Screen**   | `Yes`              | Show metrics on OLED              |
| **Display Fahrenheit**          | `No`               | Metric system (°C)                |

### Canned Messages

| Parameter          | Value                                       | Note                                    |
| ------------------ | ------------------------------------------- | --------------------------------------- |
| **Enabled**        | `Yes`                                       | Convenient for quick replies            |
| **Messages**       | `Test link,On air,73!,Coordinates received` | Comma-separated, no spaces after        |
| **Send Bell**      | `No`                                        | Don't send bell character with messages |
| **Rotary Encoder** | `No`                                        | If no encoder connected                 |

### Disabled Modules (Recommendation for OneMesh)

| Module                    | State | Why                                |
| ------------------------- | ----- | ---------------------------------- |
| **Serial**                | `No`  | Console not used                   |
| **External Notification** | `No`  | Without external piezo/LED modules |
| **Store & Forward**       | `No`  | Only for `REPEATER`/`ROUTER`       |
| **Range Test**            | `No`  | Creates excess traffic             |
| **Audio**                 | `No`  | Not supported on Heltec V4         |
| **Remote Hardware**       | `No`  | Not used                           |
| **Ambient Lighting**      | `No`  | Save power                         |
| **Detection Sensor**      | `No`  | Without external GPIO sensors      |
| **Paxcounter**            | `No`  | Excess traffic for OneMesh         |
| **Status Message**        | `No`  | Not needed for stationary node     |

---

## 🔑 Critical Flags and Application Order

1. **Channel Name**: Primary must be **exactly** `LongFast`.
2. **PSK**: `AQ==` in primary channel is required for OneMesh filtering.
3. **OK to MQTT**: `On` - without this, neighbors can't relay your packets to server.
4. **Save Order** (UI bug workaround):
   - LoRa → Region → save
   - Channels → name + PSK + Uplink/Downlink → save
   - MQTT → basic params → save
   - MQTT → Map Reports → save separately
   - Position / Device / Power → save
   - Reboot node
5. **Verification**:
   - In app: MQTT → status `Connected`
   - On map: node appears within 1–3 hours
   - In `LongFast` chat: messages from neighbors visible

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

## 🆘 Help and Community

- 🗺️ [ONEmesh Map](https://map.onemesh.ru) - network visualization
- 💬 [Telegram Group](https://t.me/onemesh_ru) - questions, help, city coordination
- 📘 [Official Meshtastic Docs](https://meshtastic.org/docs/)
- 🐙 [Firmware Repository](https://github.com/meshtastic/firmware)
- 🔧 [Heltec V4 Specs](https://meshtastic.org/docs/hardware/devices/Heltec-automation/lora32/?Heltec=v4)
