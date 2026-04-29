---
author: ["Potato Energy Team", "ponfertato"]
categories: ["smart-home", "home-assistant", "configuration", "guide"]
date: "2026-04-29T12:20:00+03:00"
description: "Precise home zone configuration in Home Assistant: coordinates, elevation, radius. Via UI and YAML."
draft: false
slug: "location-config"
tags: ["home-assistant", "configuration", "gps", "zone", "yaml"]
title: "Home Assistant: Precise Home Location Setup"
---

The home zone (`zone.home`) is the foundation for:

- ✅ Presence detection (automations for "arrived/left")
- ✅ Sunrise/sunset calculation (lighting, blinds)
- ✅ Weather forecasting (coordinate-based)
- ✅ Geofencing for devices and users

Inaccurate coordinates = false triggers, wrong forecasts, broken automations.

---

## 🎯 The Default Wizard Problem

During initial setup, Home Assistant asks to pick a location on a map. But:

- ❌ No manual coordinate input in the wizard
- ❌ IP-based auto-detection often has 1–10 km error
- ❌ Elevation above sea level is not requested

**Solution**: configure precise coordinates after installation, via UI or YAML.

---

## 🔧 Method 1: Via UI (After Installation)

### Step 1: Get Precise Coordinates

Use a GPS-enabled device (phone with Home Assistant app):

1. Install the [official companion app](https://companion.home-assistant.io/)
2. Grant precise location permission
3. Enable `device_tracker.<device>` sensor in app settings
4. Open **Developer Tools → States** in web UI
5. Find entity `device_tracker.<your_device>`
6. Copy attributes: `latitude`, `longitude`, `altitude`

### Step 2: Update Home Zone

1. **Settings → Areas, labels & zones → Zones**
2. Open `home`
3. Paste coordinates into `Latitude` / `Longitude` fields
4. Save

### Step 3: Set Elevation

1. **Settings → System → General**
2. Fill `Elevation above sea level` (in meters)
3. Save

> 💡 **Zone radius**: default is 100 m. Increase to 200–500 m for rural areas or if GPS has noticeable error.

---

## ⚙️ Method 2: Via YAML (Advanced)

Add to `configuration.yaml`:

```yaml
homeassistant:
  name: Home
  latitude: 55.7558 # ← example, replace with yours
  longitude: 37.6173 # ← example, replace with yours
  elevation: 156 # elevation in meters
  radius: 200 # zone radius in meters
  unit_system: metric
  time_zone: "Europe/Moscow"
  country: "RU"
  currency: "RUB"
```

**Important**:

- After changing `configuration.yaml`, run **Check Configuration** and **Restart**
- If coordinates are set in YAML, UI fields become read-only

---

## 🔄 Alternative: `homeassistant.set_location` Action

For dynamic updates (e.g., from automation):

```yaml
automation:
  - alias: "Update home location"
    trigger:
      platform: time_pattern
      minutes: "/30" # every 30 minutes
    action:
      - action: homeassistant.set_location
        data:
          latitude: "{{ states('sensor.gps_latitude') }}"
          longitude: "{{ states('sensor.gps_longitude') }}"
          elevation: "{{ states('sensor.gps_altitude') }}"
```

> ⚠️ Use with caution: frequent location updates may break presence automations.

---

## ⚠️ Common Issues

```yaml
# "Away" doesn't change to "Home" on return
→ Check zone radius: increase to 300–500 m
→ Ensure `device_tracker` is updating (interval, min distance)

# Wrong sunrise/sunset time
→ Verify `elevation` and `time_zone` in settings
→ Reload `sun` integration

# Coordinates won't save in UI
→ They may be set in `configuration.yaml` - edit only there
```

---

## Links

- 🏠 [Home Assistant Location Docs](https://www.home-assistant.io/integrations/homeassistant/#general-settings)
- 📍 [Zone Configuration](https://www.home-assistant.io/integrations/zone/)
- 📱 [Companion App Location Tracking](https://companion.home-assistant.io/docs/core/location)
- 🗺️ [Get Coordinates (OpenStreetMap)](https://www.openstreetmap.org)
