---
author: ["Potato Energy Team", "ponfertato"]
categories: ["mesh", "home-assistant", "mqtt", "telegram", "guide"]
date: "2026-04-29T113:15:00+03:00"
description: "Meshtastic + Home Assistant + ONEmesh: MQTT integration, Telegram notifications, node monitoring. Part 3: Integrations."
draft: false
series: ["Mesh Networks"]
slug: "integrations"
tags: ["meshtastic", "home-assistant", "mqtt", "telegram", "onemesh", "yaml"]
title: "Heltec V4: Integrations. Home Assistant, Telegram, Monitoring. Part 3"
---

> 📌 **This is Part 3 of the series**. [Part 1: Theory]({{< ref "/blog/mesh/heltec-v4-mesh-intro.en.md" >}}), [Part 2: Practice]({{< ref "/blog/mesh/heltec-v4-practice.en.md >}}).

---

## 🔗 What We're Integrating

| Component          | Purpose                                                           | Difficulty |
| ------------------ | ----------------------------------------------------------------- | ---------- |
| **ONEmesh MQTT**   | Bridge between local mesh and global map + Telegram notifications | 🟢 Low     |
| **Home Assistant** | Node sensors, device_tracker, automations on mesh messages        | 🟡 Medium  |
| **Notifications**  | Alerts for new nodes, lost connection, mentions in chat           | 🟢 Low     |

> 💡 All examples tested on: HA 2026.4.2 (Docker) + Orange Pi 3B + built-in MQTT broker.

---

## 🌐 Step 1: Configure Meshtastic MQTT for ONEmesh

### Connection Parameters

| Parameter             | Value             | Note                                                         |
| --------------------- | ----------------- | ------------------------------------------------------------ |
| **MQTT Address**      | `mqtt.onemesh.ru` | Community server                                             |
| **Username**          | `onemesh`         | Default                                                      |
| **Password**          | `onecat`          | Shared for all                                               |
| **Root topic**        | `msh/RU/XXX`      | `XXX` = city code ([list](https://map.onemesh.ru/mqtt.html)) |
| **TLS enabled**       | `Yes`             | Try `No` if not working                                      |
| **JSON enabled**      | `Yes`             | Required for Home Assistant                                  |
| **Uplink / Downlink** | `Yes` / `No`      | Uplink only is enough for monitoring                         |

> ⚠️ **Important**: Firmware 2.5+ supports new encryption key format (PKI). If using encryption - ensure keys are synced across nodes.

### How to Verify

1. Enable `JSON enabled` in MQTT settings
2. Connect to broker via `mosquitto_sub` or MQTT Explorer:

```bash
mosquitto_sub -h mqtt.onemesh.ru -u onemesh -P onecat \
  -t 'msh/RU/MOW/json/#' -v
```

3. You'll see messages like:

```json
{
  "id": 452664778,
  "from": 2130636288,
  "payload": {
    "text": "Hello from local mesh!",
    "battery_level": 87,
    "voltage": 4.12
  },
  "type": "text",
  "timestamp": 1714234567
}
```

---

## 📬 Step 2: Telegram Notifications via ONEmesh Bot

ONEmesh automatically forwards messages from your city topic to a private chat with the bot.

### Notification Examples

```
[26.04.2026 20:52] ONEmesh Bot: LF | qwrt (https://map.onemesh.ru/?node_id=483535916)  QWRT Pocket
20, standard. but for home node - triad antenna
SNR 4.75  RSSI -92  HopLimit 3
- data from 44b4 (https://map.onemesh.ru/?node_id=49169588)
```

| Field      | Meaning                  | Why it matters                     |
| ---------- | ------------------------ | ---------------------------------- |
| `SNR`      | Signal-to-Noise Ratio    | Signal quality: higher = better    |
| `RSSI`     | Received Signal Strength | Signal power: closer to 0 = better |
| `HopLimit` | Remaining hops           | Shows how far message traveled     |

### Setup

1. Find [@ONEmeshBot](https://t.me/ONEmeshBot) in Telegram
2. Send `/start` - bot will show instructions
3. Ensure your node sends data to MQTT with correct `root topic`
4. Done: notifications will arrive automatically

> 💡 **Tip**: if no notifications - check that `OK to MQTT` is enabled in channel settings and node has internet.

---

## 🏠 Step 3: Home Assistant Integration

### Prerequisites

- [ ] Built-in MQTT broker in HA (`Settings → Devices & Services → MQTT`)
- [ ] Meshtastic node with `JSON enabled` in MQTT
- [ ] Access to `configuration.yaml` and ability to create `mqtt.yaml`

### Step 3.1: Connect MQTT in HA

```yaml
# configuration.yaml
mqtt: !include mqtt.yaml
```

```yaml
# mqtt.yaml
broker: core-mosquitto # or your broker address
port: 1883
username: homeassistant
password: !secret mqtt_password
# For external broker (e.g., ONEmesh):
# broker: mqtt.onemesh.ru
# port: 8883
# username: onemesh
# password: onecat
# certificate: /config/certs/ca.crt  # for TLS
```

### Step 3.2: Create Sensors for Node

```yaml
# mqtt.yaml - continued
sensor:
  # Node battery
  - name: "Meshtastic Node Battery"
    unique_id: "meshtastic_node_battery"
    state_topic: "msh/RU/MOW/json/LongFast/!abcd1234"
    value_template: >-
      {% if value_json.from == 2130636288 and value_json.payload.battery_level is defined %}
        {{ value_json.payload.battery_level | int }}
      {% else %}
        {{ this.state }}
      {% endif %}
    unit_of_measurement: "%"
    device_class: battery

  # Voltage
  - name: "Meshtastic Node Voltage"
    unique_id: "meshtastic_node_voltage"
    state_topic: "msh/RU/MOW/json/LongFast/!abcd1234"
    value_template: >-
      {% if value_json.from == 2130636288 and value_json.payload.voltage is defined %}
        {{ value_json.payload.voltage | float | round(2) }}
      {% else %}
        {{ this.state }}
      {% endif %}
    unit_of_measurement: "V"
    device_class: voltage

  # Channel utilization (airtime load)
  - name: "Meshtastic Channel Utilization"
    unique_id: "meshtastic_chutil"
    state_topic: "msh/RU/MOW/json/LongFast/!abcd1234"
    value_template: >-
      {% if value_json.from == 2130636288 and value_json.payload.channel_utilization is defined %}
        {{ value_json.payload.channel_utilization | float | round(1) }}
      {% else %}
        {{ this.state }}
      {% endif %}
    unit_of_measurement: "%"
```

> 🔑 **Important**: replace `2130636288` with your node's decimal `from` (find via MQTT Explorer or `meshtastic --info`).

### Step 3.3: Location Tracking (device_tracker)

```yaml
# automations.yaml
- alias: "Meshtastic: Update node location"
  trigger:
    platform: mqtt
    topic: "msh/RU/MOW/json/LongFast/!abcd1234"
    value_template: >-
      {% if value_json.from == 2130636288 and 
            value_json.payload.latitude_i is defined and 
            value_json.payload.longitude_i is defined %}on{% endif %}
  action:
    service: device_tracker.see
    data:
      dev_id: "meshtastic_node_1"
      gps:
        - "{{ (trigger.payload_json.payload.latitude_i | int * 1e-7) }}"
        - "{{ (trigger.payload_json.payload.longitude_i | int * 1e-7) }}"
      battery: "{{ states('sensor.meshtastic_node_battery') | default(0) }}"
```

> 💡 Meshtastic stores coordinates as `integer * 1e7`, so multiply by `1e-7` to get real values.

### Step 3.4: Notifications on Chat Mentions

```yaml
# automations.yaml
- alias: "Meshtastic: Notify on mention"
  trigger:
    platform: mqtt
    topic: "msh/RU/MOW/json/LongFast/!abcd1234"
    value_template: >-
      {% if value_json.type == "text" and 
            value_json.payload.text is defined and 
            "ponf" in value_json.payload.text.lower() %}on{% endif %}
  action:
    service: notify.mobile_app_your_phone
    data:
      title: "📡 Meshtastic"
      message: "Mention: {{ trigger.payload_json.payload.text }}"
      data:
        priority: high
```

---

## 🔐 Security: Encryption and Keys

### What Changed in Firmware 2.5+

| Version   | Key Type                 | Notes                                           |
| --------- | ------------------------ | ----------------------------------------------- |
| **< 2.5** | Single `PSK` per channel | Simple setup, but vulnerable if compromised     |
| **≥ 2.5** | `PKI` + `shared secret`  | Separate keys for encryption and authentication |

### How to Configure Encryption

1. In Meshtastic app: **Channel → Primary → PSK**
   - Enable encryption
   - Copy the key (same `AQ==` for primary channel)
2. In MQTT settings: **Encryption enabled → Yes**
3. Ensure all nodes in mesh use the same key

> ⚠️ **Critical**: if keys don't match - messages arrive but won't decrypt. Verify via `MQTT Explorer`.

### Where to Store Keys

- ✅ In `secrets.yaml` Home Assistant (don't commit to Git!)
- ✅ In password manager (Bitwarden, KeePassXC)
- ❌ Not in public repos, logs, or screenshots

---

## ⚠️ Common Issues

| Symptom                     | Cause                            | Fix                                               |
| --------------------------- | -------------------------------- | ------------------------------------------------- |
| Sensors not updating        | Wrong `from` in `value_template` | Find node's decimal ID via `meshtastic --info`    |
| No Telegram messages        | Wrong `root topic`               | Check city code: `msh/RU/MOW`, `msh/RU/SPB`, etc. |
| Encryption not working      | Keys don't match                 | Copy `PSK` exactly, no spaces, to both nodes      |
| `device_tracker` not moving | Coordinates in wrong format      | Multiply `latitude_i`/`longitude_i` by `1e-7`     |
| HA won't connect to broker  | Wrong credentials                | Check `username`/`password` and port (1883/8883)  |

---

## Links

- 📡 [Meshtastic MQTT Docs](https://meshtastic.org/docs/configuration/module/mqtt/)
- 🏠 [HA Meshtastic Integration](https://meshtastic.org/docs/software/home-assistant/)
- 🗺️ [ONEmesh Map](https://map.onemesh.ru)
- 🔐 [Meshtastic Security Guide](https://meshtastic.org/docs/configuration/security/)
