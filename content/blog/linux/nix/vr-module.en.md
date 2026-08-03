---
author: ["Potato Energy Team", "ponfertato"]
categories: ["nix", "linux", "vr", "hardware"]
date: "2026-08-03T09:00:00+03:00"
description: "VR setup on NixOS for Meta Quest 3S: WiVRn, WayVR, OpenComposite, and troubleshooting Steam sandbox issues."
draft: false
series: ["Nix/NixOS"]
slug: "vr-wivrn-wayvr"
tags: ["nixos", "wivrn", "wayvr", "opencomposite", "steam", "quest3s"]
title: "NixOS: Complete VR Stack Setup (WiVRn + WayVR)"
---

Running VR on NixOS requires an understanding of the interaction between OpenXR runtimes, compositors, and the Steam sandbox. For the **Meta Quest 3S + Steam (Proton)** setup, the optimal and most stable solution is using **WiVRn** as the streamer and OpenXR runtime, with optional **OpenComposite** for backward compatibility with older OpenVR titles.

> 💡 **Critically important:** VR packages in NixOS evolve rapidly. It is highly recommended to use packages from the `nixos-unstable` branch or the [nixpkgs-xr](https://github.com/nix-community/nixpkgs-xr) overlay.

---

## 📦 Base NixOS Configuration

Create or update your module (e.g., `modules/vr.nix`). This configuration solves two major NixOS VR issues: async reprojection (requiring `CAP_SYS_NICE`) and Steam isolation (Pressure Vessel).

```nix
{ config, lib, pkgs, ... }:
{
  # 1. Load kernel module for virtual input (critical for WayVR)
  boot.kernelModules = [ "uinput" ];

  # 2. Add user to required groups
  users.users.ponfertato.extraGroups = [ "input" "plugdev" "adbusers" ];

  services.wivrn = {
    enable = true;
    autoStart = true;
    openFirewall = true;

    # Solves stuttering issues (async reprojection requires CAP_SYS_NICE)
    highPriority = true;

    # Automatically injects PRESSURE_VESSEL_IMPORT_OPENXR_1_RUNTIMES=1
    # This allows sandboxed Steam games to see the WiVRn OpenXR runtime
    steam = {
      enable = true;
      importOXRRuntimes = true;
    };

    # Environment variables for proper screen capture in KDE Wayland
    monadoEnvironment = {
      XDG_CURRENT_DESKTOP = "KDE";
      XDG_SESSION_TYPE = "wayland";
      QT_QPA_PLATFORM = "wayland";
    };

    # Optional: auto-launch WayVR from the headset menu
    config = {
      enable = true;
      json = {
        application = [ pkgs.unstable.wayvr ];
      };
    };
  };

  environment.systemPackages = with pkgs.unstable; [
    wayvr         # VR desktop overlay
    opencomposite # OpenVR -> OpenXR compatibility layer
    android-tools # For ADB and wired pairing if needed
  ];
}
```

---

## ⚠️ Troubleshooting Common Issues

### 1. WayVR shows only Pass-through camera

If WayVR launches but does not render the UI, and logs show `uinput` or `OpenXR-Loader` errors, perform these checks:

- **`input` group:** Ensure the user is added to the `input` group (as shown in the config above) and you have **rebooted** or re-logged into the session afterward. Without this, WayVR cannot initialize virtual input devices and blocks UI rendering.
- **Launch order:** Launch WayVR strictly _after_ the headset is connected to WiVRn.

  ```bash
  # 1. Ensure the service is running
  systemctl --user status wivrn

  # 2. If auto-start failed, launch via the Steam wrapper
  # (this resolves NixOS FHS socket access issues)
  steam-run wayvr --replace
  ```

- **iGPU limitations:** On older integrated graphics (e.g., Intel HD 530), Vulkan compositing in WayVR may be unstable. If the UI does not appear, fall back to the built-in **SteamVR Desktop** (launch SteamVR from the WiVRn menu and select "Desktop").

### 2. Steam games do not detect the OpenXR runtime

Even with `importOXRRuntimes = true` enabled, some games may require explicit declaration. If a game crashes or runs in flat mode, add this to its Steam launch options:

```text
PRESSURE_VESSEL_IMPORT_OPENXR_1_RUNTIMES=1 %command%
```

For older games strictly requiring OpenVR, use OpenComposite:

```text
env XR_RUNTIME_JSON=/run/current-system/sw/share/openxr/1/opencomposite_runtime.json %command%
```

### 3. Error: `failed to determine active runtime file path`

The OpenXR loader cannot find the manifest. WiVRn manages this automatically, but if you use custom settings, verify the file exists:

```bash
ls -l /run/current-system/sw/share/openxr/1/openxr_wivrn.json
```

If using Home Manager, you can explicitly create the symlink:

```nix
xdg.configFile."openxr/1/active_runtime.json".source = "${pkgs.unstable.wivrn}/share/openxr/1/openxr_wivrn.json";
```

---

## 🔄 Connection Process (Quest 3S)

1. Enable **Developer Mode** in the Meta Horizon app.
2. Connect the headset via USB-C. Approve debugging in the headset prompt.
3. Install the WiVRn APK (download the latest release from GitHub):
   ```bash
   adb install WiVRn-vX.Y.Z.apk
   ```
4. Disconnect USB. Ensure the PC and headset are on the same Wi-Fi network (5 GHz preferred).
5. Launch the WiVRn app on the headset. It will automatically discover the PC via Avahi (mDNS).
6. In the headset menu, select **WayVR** (or SteamVR, if WayVR fails to render).
