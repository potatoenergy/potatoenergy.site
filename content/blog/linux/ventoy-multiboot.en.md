---
author: ["Potato Energy Team", "ponfertato"]
categories: ["linux", "ventoy", "multiboot", "guide"]
date: "2026-05-06T12:35:00+03:00"
description: "Ventoy: one USB drive - dozens of images. Organization, themes, Windows auto-install."
draft: false
series: ["Linux"]
slug: "ventoy-multiboot"
tags: ["ventoy", "multiboot", "windows", "linux", "automation", "grub"]
title: "Ventoy: Multi-Boot USB with Correct Structure"
---

Instead of reformatting USB for each new image, **Ventoy** lets you simply copy ISO files like regular files. At boot, you get a menu with all available images.

**Benefits**:

- ✅ No need to rewrite USB for each image
- ✅ Support for Windows, Linux, utilities - all on one drive
- ✅ Regular files accessible from any OS
- ✅ Flexible configuration via JSON

---

## 🗂 Correct USB Structure

```
[First USB partition - exFAT/NTFS]
├── /ventoy/                    # ← Must be here!
│   ├── ventoy.json            # Main config
│   ├── revi/                  # Windows auto-install
│   │   └── autounattend.xml
│   └── theme/                 # Custom theme
│       └── distro/theme.txt
│
├── BACKUP/                    # Working files
├── LINUX/                     # Linux ISOs
│   ├── Archlinux 2025.12.01.iso
│   ├── Debian 13.2.0.iso
│   └── NixOS 25.11.1734.iso
├── WINDOWS/                   # Windows ISOs
│   ├── Windows 10 22H2.iso
│   ├── Windows 10 Enterprise LTSC 2021.iso
│   └── Windows 11 25H2.iso
├── ReviSetup/                 # Post-install scripts
│   └── setup.cmd
└── UTILS/                     # Utilities
    ├── gparted-live.iso
    └── memtest86+.iso
```

> ⚠️ **Critical**: `/ventoy/` must be on the **first partition** (where ISOs are), NOT in root!

---

## ⚙️ Basic Configuration: `ventoy.json`

File must be at `/ventoy/ventoy.json`, UTF-8 encoding.

### Minimal Example

```json
{
  "control": [
    { "VTOY_DEFAULT_MENU_MODE": "1" },
    { "VTOY_FILT_DOT_UNDERSCORE_FILE": "1" }
  ],
  "theme": {
    "file": "/ventoy/theme/distro/theme.txt",
    "gfxmode": "1920x1080",
    "resolution_fit": "1"
  }
}
```

| Parameter                       | Description                          |
| ------------------------------- | ------------------------------------ |
| `VTOY_DEFAULT_MENU_MODE`        | Default menu mode                    |
| `VTOY_FILT_DOT_UNDERSCORE_FILE` | Hide files starting with `.` and `_` |
| `theme.file`                    | Path to GRUB theme file              |
| `gfxmode`                       | Menu resolution                      |
| `resolution_fit`                | Auto-fit to screen                   |

---

## 🎨 Customization: Menu Themes

### Where to Get Themes

1. **[distro-grub-themes](https://github.com/AdisonCavani/distro-grub-themes)** - collection of ready-made themes
2. **[Gnome-look.org](https://www.gnome-look.org/browse?cat=109)** - themes tagged `ventoy`
3. **Create your own** - follow Ventoy documentation

### Installing a Theme

```bash
# Clone theme
cd /mnt/ventoy/ventoy/theme
git clone https://github.com/AdisonCavani/distro-grub-themes.git distro

# Or download from Gnome-look
wget https://www.gnome-look.org/.../theme.tar.gz
tar xzf theme.tar.gz
```

In `ventoy.json` specify path:

```json
{
  "theme": {
    "file": "/ventoy/theme/distro/theme.txt",
    "resolution_fit": "1"
  }
}
```

---

## 🪟 Windows Auto-Install

### Structure for Auto-Install

```
/ventoy/
└── revi/
    └── autounattend.xml     # Windows Setup的 answers

/ReviSetup/
└── setup.cmd                # Post-install
```

### Example `ventoy.json` for Auto-Install

```json
{
  "auto_install": [
    {
      "parent": "/WINDOWS",
      "template": ["/ventoy/revi/autounattend.xml"],
      "autosel": 1
    }
  ]
}
```

| Parameter  | Value                      |
| ---------- | -------------------------- |
| `parent`   | Folder with Windows images |
| `template` | Path to `autounattend.xml` |
| `autosel`  | Auto-select template       |

### What `autounattend.xml` Does

- Skips Microsoft account creation
- Disables telemetry
- Applies privacy settings
- Runs `setup.cmd` after install

> 💡 Ready-made configs: [meetrevision/ventoy-conf](https://github.com/meetrevision/ventoy-conf)

---

## 🛠 Post-Install: `setup.cmd`

Script runs automatically after Windows installation.

### Example Actions

```cmd
:: Disable driver updates
reg add "HKLM\Software\Policies\Microsoft\Windows\DriverSearching" /v "SearchOrderConfig" /t REG_DWORD /d 0 /f

:: Pause updates until 2038
reg add "HKLM\SOFTWARE\Microsoft\WindowsUpdate\UX\Settings" /v "PauseUpdatesExpiryTime" /t REG_SZ /d "2038-01-19T03:14:07Z" /f

:: Disable telemetry
reg add "HKLM\Software\Policies\Microsoft\Windows\DataCollection" /v "AllowTelemetry" /t REG_DWORD /d 0 /f

:: Prevent Teams auto-install
reg add "HKLM\SOFTWARE\Policies\Microsoft\Windows\Windows Chat" /v "ChatIcon" /t REG_DWORD /d 2 /f
```

---

## 🗂 Checklist: Creating USB

- [ ] Installed Ventoy via `Ventoy2Disk.sh` / `.exe`
- [ ] Created `/ventoy/` on first partition
- [ ] Placed `ventoy.json` in `/ventoy/` (UTF-8)
- [ ] Copied images to logical folders (`LINUX/`, `WINDOWS/`)
- [ ] (Optional) Installed menu theme
- [ ] (Optional) Configured Windows auto-install
- [ ] Verified JSON syntax via [json.cn](https://www.json.cn)
- [ ] Tested boot

---

## 🔧 Useful Commands

```bash
# Check ventoy.json syntax
cat /ventoy/ventoy.json | jq .

# In Ventoy menu: F5 - show ventoy.json content

# Recreate Ventoy partition (data will be lost!)
sudo ./Ventoy2Disk.sh -i /dev/sdX
```

---

## 📚 Additional Features

Ventoy supports many plugins and extensions:

- **Persistence** - save data between Live system reboots
- **Memdisk** - boot images into RAM
- **Auto install** - OS installation automation
- **Custom menu** - custom menu items
- **Injection** - inject drivers/files into images

> 🔗 **Documentation**: [ventoy.net](https://www.ventoy.net/en/plugin_entry.html)

---

## ⚠️ Common Issues

| Symptom                   | Solution                                   |
| ------------------------- | ------------------------------------------ |
| `ventoy.json` not applied | Check path: strictly `/ventoy/ventoy.json` |
| Theme not loading         | Check encoding (UTF-8) and path in config  |
| Auto-install not working  | Ensure `parent` points to correct folder   |
| JSON parse error          | Check syntax via online validator          |

---

## 🔐 Security

- ✅ exFAT - accessible from any OS (but unencrypted)
- 🔐 For sensitive files: VeraCrypt container
- ✅ `autounattend.xml` and `setup.cmd` - plain text, no passwords

---

## 🗂 Links

- 🚀 [Official Ventoy Site](https://www.ventoy.net)
- 🎨 [Theme Collection](https://github.com/AdisonCavani/distro-grub-themes)
- 🎨 [Themes on Gnome-look](https://www.gnome-look.org/browse?cat=109&tag=ventoy)
- 🪟 [Windows Auto-Install](https://github.com/meetrevision/ventoy-conf)
- 📘 [Plugin Documentation](https://www.ventoy.net/en/plugin_entry.html)
