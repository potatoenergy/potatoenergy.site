---
author: ["Potato Energy Team", "ponfertato"]
categories: ["nix", "linux", "containers"]
date: "2026-03-18T17:25:00+03:00"
description: "Run Ubuntu/Debian apps on NixOS via Distrobox. Container setup, install, launcher, full cleanup."
draft: false
series: ["Nix/NixOS"]
slug: "distrobox-apps"
tags: ["nix", "distrobox", "containers", "desktop"]
title: "NixOS: Running Foreign Apps via Distrobox"
---

Sometimes you need to run an app on NixOS that's only available for Ubuntu/Debian or ships as `.deb`/`.iso`. Distrobox lets you run it in a container with host integration - like a native app.

> 💡 Works on any distro with Nix installed.

---

## 📦 Create container

```bash
# Create container with necessary volume mounts
nix shell nixpkgs#distrobox --command distrobox create \
  --image ubuntu:20.04 \
  --name <container-name> \
  --volume "<volume_pts>:/dev/pts" \
  --volume "<volume_journal>:/var/log/journal" \
  --home "$HOME/<container-name>-home"
```

**Parameters**:

- `--image` - base image (ubuntu:20.04, debian:11, etc.)
- `--volume` - mount directories (audio, logs)
- `--home` - separate home directory inside container
- `--name` - container name (used in other commands)

**Enter container**:

```bash
nix shell nixpkgs#distrobox --command distrobox enter <container-name>
```

---

## 🔧 Install app inside container

```bash
# Inside container (after distrobox enter)
sudo sh -c '
  # Clean old configs
  rm -rf /etc/apt/sources.list.d/<vendor>.list*
  umount /mnt/<vendor> 2>/dev/null || true

  # Download and mount image
  wget -O /tmp/<vendor>.iso "https://<url>/<vendor>.iso"
  mkdir -p /mnt/<vendor>
  mount -o loop,ro -t iso9660 /tmp/<vendor>.iso /mnt/<vendor>

  # Install from local repo
  echo "deb [trusted=yes] file:/mnt/<vendor>/repo ./" > /etc/apt/sources.list.d/<vendor>.list
  apt update
  apt install -y <package-name>-full

  # Cleanup
  umount /mnt/<vendor>
  rm -f /etc/apt/sources.list.d/<vendor>.list /tmp/<vendor>.iso
'
```

**Replace placeholders**:
| `<vendor>` | Vendor name (e.g., `nausoftphone`) |
| `<package-name>` | Package name in repo |
| `<url>` | URL to installer image |

**Verify**:

```bash
<package-name> --version
```

---

## 🖥 Desktop launcher (.desktop)

```desktop
# ~/.local/share/applications/<app>-distrobox.desktop
[Desktop Entry]
Version=1.0
Type=Application
Name=<App Name> (Distrobox)
Comment=Run <App Name> inside Distrobox container
Exec=konsole --hold -e bash -c 'nix shell nixpkgs#distrobox --command distrobox enter --name <container-name> -- <app-command>'
Icon=<icon-name>
Terminal=false
Categories=Network;AudioVideo;
Keywords=<app>;distrobox;container;
```

**Apply**:

```bash
update-desktop-database ~/.local/share/applications/
```

App now appears in menu, launchers, and system search.

---

## 🔄 Update app

```bash
# Enter container
nix shell nixpkgs#distrobox --command distrobox enter <container-name>

# Upgrade package
sudo apt update && sudo apt install --only-upgrade <package-name>-full
```

---

## 🗑 Full cleanup (container + volumes)

```bash
# 1. Remove container
distrobox rm <container-name>

# 2. Remove associated volumes (if created separately)
# For Podman:
podman volume rm <volume_pts> <volume_journal>

# For Docker:
docker volume rm <volume_pts> <volume_journal>

# 3. Remove container home directory (if used)
rm -rf "$HOME/<container-name>-home"

# 4. Remove launcher
rm -f ~/.local/share/applications/<app>-distrobox.desktop
update-desktop-database ~/.local/share/applications/
```

> ⚠️ Volume deletion is irreversible - ensure no needed data inside.

**Verify cleanup**:

```bash
# Containers
distrobox list | grep <container-name>

# Volumes (Podman)
podman volume ls | grep <volume_pts>

# Volumes (Docker)
docker volume ls | grep <volume_pts>
```

---

## ⚙️ Launch optimization

### Hide terminal (for production)

```desktop
# Replace Exec with:
Exec=sh -c 'nix shell nixpkgs#distrobox --command distrobox enter --name <container-name> -- <app-command>' >> ~/.cache/<app>.log 2>&1 &
```

### Fix audio (if not working)

```bash
# Add when creating container:
--volume "$XDG_RUNTIME_DIR/pulse:/run/user/1000/pulse"
# Or for PipeWire:
--volume "$XDG_RUNTIME_DIR/pipewire-0:/run/user/1000/pipewire-0"
```

---

## ⚠️ Common issues

```bash
# Launcher not appearing
→ Validate syntax: desktop-file-validate ~/.local/share/applications/<app>-distrobox.desktop
→ Refresh cache: update-desktop-database ~/.local/share/applications/

# Container won't start after Nix update
→ Recreate: distrobox rm <name> && create anew
→ Data in --home persists if directory not manually deleted

# No audio/device access
→ Check volume mounts during container creation
→ Ensure user is in required groups (audio, input)

# Volume won't delete ("in use")
→ Ensure container is stopped: distrobox list
→ Force stop: podman stop <container-name> || docker stop <container-name>
```

---

## Links

- 🐧 [Distrobox Docs](https://distrobox.it/)
- 🦊 [NixOS Wiki: Distrobox](https://wiki.nixos.org/wiki/Distrobox)
- 🖥 [Desktop Entry Spec](https://specifications.freedesktop.org/desktop-entry-spec/)
