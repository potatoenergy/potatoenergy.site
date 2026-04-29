---
author: ["Potato Energy Team", "ponfertato"]
categories: ["home-assistant", "docker", "bluetooth", "guide"]
date: "2026-04-29T12:45:00+03:00"
description: "Home Assistant in Docker: Bluetooth setup for device discovery. D-Bus passthrough, passive scanning, Meshtastic integration."
draft: false
series: ["Home Assistant"]
slug: "bluetooth-docker"
tags: ["home-assistant", "docker", "bluetooth", "meshtastic", "ble"]
title: "Home Assistant + Docker: Bluetooth for Device Discovery"
---

Home Assistant in Docker doesn't see Bluetooth devices, even if everything works on the host.

**Causes**:

- ❌ Container has no direct access to `/dev/hci0`
- ❌ BlueZ inside container conflicts with host daemon
- ❌ Passive BLE scanning requires `--experimental` flag in BlueZ

**Solution**: Don't run Bluetooth stack inside container - passthrough D-Bus from host instead.

---

## ✅ Docker Compose Configuration

### Minimal Setup

```yaml
services:
  home-assistant:
    container_name: home-assistant
    image: ghcr.io/home-assistant/home-assistant:stable
    volumes:
      - config:/config
      - /run/dbus:/run/dbus:ro # ← Critical for Bluetooth
    cap_add:
      - NET_ADMIN
      - NET_RAW
      - SYS_ADMIN
    restart: unless-stopped
    networks:
      - traefik
      - prometheus

volumes:
  config:
    driver: local

networks:
  traefik:
    external: true
    name: traefik
  prometheus:
    external: true
    name: prometheus
```

> ⚠️ **Don't add `devices: - /dev/hci0:/dev/hci0`** - not needed with D-Bus passthrough and may cause conflicts.

---

## 🔧 BlueZ Configuration on Host

### Enabling Experimental Mode (for Passive Scanning)

Home Assistant uses passive BLE scanning, which requires running `bluetoothd` with `--experimental` flag.

```bash
# 1. Create systemd override
sudo systemctl edit bluetooth

# 2. Insert:
[Service]
ExecStart=
ExecStart=/usr/lib/bluetooth/bluetoothd --experimental

# 3. Apply and restart
sudo systemctl daemon-reload
sudo systemctl restart bluetooth

# 4. Verify flag is applied
ps aux | grep bluetoothd
# Expected: /usr/lib/bluetooth/bluetoothd --experimental
```

> 💡 **Why this is needed**: Passive scanning doesn't send active requests, saving device battery. But this feature in BlueZ is marked "experimental".

---

## 🔗 Pairing Devices (On Host Only!)

Manage Bluetooth **only on the host**, not inside the container.

```bash
# 1. Enable adapter
sudo bluetoothctl power on

# 2. Start scanning
sudo bluetoothctl scan on

# 3. When device appears (e.g., Meshtastic_XXXX):
sudo bluetoothctl pair AA:BB:CC:DD:EE:FF
sudo bluetoothctl trust AA:BB:CC:DD:EE:FF
sudo bluetoothctl connect AA:BB:CC:DD:EE:FF
```

> ✅ After pairing on host, Home Assistant will see the device via passthrough D-Bus.

---

## ⚙️ Integration in Home Assistant

### Installing Integration

1. Install [Meshtastic via HACS](https://github.com/meshtastic/meshtastic-ha) (or other needed integration)
2. Restart HA: `docker compose restart home-assistant`
3. Settings → Integrations → Add Integration → [Name]
4. Device should appear in discovered list (by name or address)

### If Device Not Discovered

```bash
# Check on host:
- Device paired? `bluetoothctl paired-devices`
- Adapter not blocked? `rfkill list`
- BlueZ running with --experimental? `ps aux | grep bluetoothd`

# In Home Assistant:
- Reload integration: Settings → System → Reload
- Check logs: Settings → System → Logs → search "bluetooth"
```

---

## ⚠️ Common Issues

| Symptom                                   | Cause                              | Solution                                  |
| ----------------------------------------- | ---------------------------------- | ----------------------------------------- |
| `Unable to open mgmt_socket` in container | Daemon conflict                    | Don't run `bluetoothctl` inside container |
| Device not appearing in scan              | Pairing mode not enabled on device | Enable pairing mode in device app         |
| `Error.Busy` on power on                  | `hciattach` holding device         | Remove `-n` flag in init service          |
| Passive scanning not working              | BlueZ without `--experimental`     | Add flag to `ExecStart` and restart       |

---

## Links

- 🏠 [Home Assistant Bluetooth Docs](https://www.home-assistant.io/integrations/bluetooth/)
- 📘 [BlueZ Experimental Features](https://www.bluez.org/experimental/)
- 🐙 [Meshtastic HA Integration](https://github.com/meshtastic/meshtastic-ha)
- 🐳 [Docker Bind Mounts](https://docs.docker.com/storage/bind-mounts/)
