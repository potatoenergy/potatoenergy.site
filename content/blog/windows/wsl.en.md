---
author: ["Potato Energy Team", "ponfertato"]
categories: ["windows", "linux", "wsl", "tutorial"]
date: "2026-03-16T22:00:00+03:00"
description: "WSL2 in practice: install, configure, integrate with Windows. Developer's guide."
draft: false
series: ["Windows Tips"]
slug: "wsl"
tags: ["windows", "wsl", "linux", "docker", "dev"]
title: "WSL2: Developer's Complete Guide"
---

WSL (Windows Subsystem for Linux) lets you run native Linux command-line tools directly on Windows - no VM, no dual boot.

**WSL1** - syscall translation layer (fast, but not 100% compatible)  
**WSL2** - real Linux kernel in lightweight virtualization (full compatibility, slightly more resources)

> 💡 Use WSL2. Near-native performance with full Docker, systemd, and Linux feature support.

---

## Requirements

- **OS**: Windows 10 (2004+, build 19041+) or Windows 11
- **Architecture**: x64 or ARM64
- **Privileges**: Administrator (for install)
- **Virtualization**: Enabled in BIOS/UEFI (Hyper-V Platform)

**Check virtualization:**

```powershell
systeminfo | findstr /I "Virtualization"
# Should show: "Hyper-V detected" or "Virtualization Enabled"
```

---

## Install (one command)

```powershell
# Run PowerShell as Administrator
wsl --install
```

This does everything:

- ✅ Enables WSL and VirtualMachinePlatform features
- ✅ Downloads and installs Ubuntu (default distro)
- ✅ Sets WSL2 as default version

**Reboot your PC** after completion.

---

## Manual install (if `--install` fails)

```powershell
# 1. Enable features
dism /online /enable-feature /featurename:Microsoft-Windows-Subsystem-Linux /all /norestart
dism /online /enable-feature /featurename:VirtualMachinePlatform /all /norestart

# 2. Reboot
shutdown /r /t 0

# 3. Download and install WSL2 kernel
# https://aka.ms/wsl2kernel

# 4. Set WSL2 as default
wsl --set-default-version 2

# 5. Install distro from Microsoft Store
# Or via winget:
winget install Ubuntu.Ubuntu.24.04
```

---

## First run

```bash
# After reboot, a terminal opens with Linux setup
# Create username and password (typing is hidden - normal)

# Update packages (Ubuntu/Debian)
sudo apt update && sudo apt upgrade -y

# Install basic tools
sudo apt install -y git curl wget build-essential
```

---

## Manage distributions

```powershell
# List installed distros
wsl --list --verbose
# Short form:
wsl -l -v

# Launch specific distro
wsl -d Ubuntu-24.04
wsl -d Debian

# Stop (terminate) a distro
wsl --terminate Ubuntu-24.04

# Stop all
wsl --shutdown

# Set version (1 or 2)
wsl --set-version Ubuntu-24.04 2

# Set default distro
wsl --set-default Ubuntu-24.04

# Export / import
wsl --export Ubuntu-24.04 D:\backups\ubuntu.tar
wsl --import MyUbuntu D:\wsl\MyUbuntu D:\backups\ubuntu.tar --version 2

# Unregister (delete) distro - all data lost!
wsl --unregister Ubuntu-24.04
```

---

## File system access

### From Linux → Windows

```bash
# Windows drives mounted under /mnt/
ls /mnt/c/Users/ponfertato/Documents

# Open Explorer in current directory
explorer.exe .
```

### From Windows → Linux

```powershell
# Open user's home in Explorer
\\wsl$\Ubuntu-24.04\home\kirill

# Or from terminal:
explorer.exe \\wsl$\Ubuntu-24.04\home\kirill
```

> ⚠️ Don't edit Linux files from Windows apps directly (via `\\wsl$`). This can corrupt metadata. Use Linux tools inside WSL.

---

## Network and ports

```bash
# Get your WSL IP
ip addr show eth0 | grep inet

# Windows and WSL share localhost (ports forward automatically)
# Start a web server in WSL:
python3 -m http.server 8000

# Accessible from Windows at:
# http://localhost:8000
```

**If ports don't forward:**

```powershell
# Check settings
wsl --status

# Enable auto-forwarding (if disabled)
# In %USERPROFILE%\.wslconfig:
[wsl2]
networkingMode=mirrored
localhostForwarding=true
```

---

## Terminal and editor integration

### Use Windows Terminal

```powershell
# Install from Microsoft Store or:
winget install Microsoft.WindowsTerminal

# WSL profiles added automatically
# Switch shells: Ctrl+Shift+1/2/3...
```

### VS Code + WSL

```bash
# Inside WSL:
code .

# Opens VS Code on Windows connected to WSL environment
# Install "Remote - WSL" extension if prompted
```

### Run Windows apps from WSL

```bash
# Launch Notepad from Linux:
notepad.exe /mnt/c/Users/ponfertato/notes.txt

# Launch PowerShell:
powershell.exe Get-Process
```

---

## Docker in WSL2

```powershell
# Install Docker Desktop for Windows
winget install Docker.DockerDesktop

# In Docker Desktop settings:
# Settings → Resources → WSL Integration → enable your distro

# Test in WSL:
docker run hello-world
```

> 💡 Docker runs natively in WSL2 - no extra setup needed inside Linux.

---

## systemd in WSL (for services)

```bash
# Enable systemd (WSL 0.67.6+)
# Create/edit: /etc/wsl.conf
[boot]
systemd=true

# Apply:
# From PowerShell:
wsl --terminate Ubuntu-24.04
wsl -d Ubuntu-24.04

# Verify:
systemctl status
```

Now services work: `systemctl start nginx`, `enable docker`, etc.

---

## Useful settings (.wslconfig)

File: `%USERPROFILE%\.wslconfig` (create if missing)

```ini
[wsl2]
# Memory: 4 GB (or 50% of RAM)
memory=4GB
# CPUs: 4 cores
processors=4
# Swap: 2 GB
swap=2GB
# Disk: dynamic, up to 256 GB
diskSize=256GB
# Network: mirrored mode (best compatibility)
networkingMode=mirrored
# Auto-shutdown: enable
autoShutdown=true
# Idle timeout: 10 minutes
idleTimeout=600000
```

Apply: `wsl --shutdown`, then restart distro.

---

## Troubleshooting

```powershell
# WSL won't start, error 0x80370102
→ Enable virtualization in BIOS
→ Check: "Control Panel" → "Programs" → "Turn Windows features on/off" → Hyper-V Platform

# "The virtual machine could not be started"
→ Run as admin:
  bcdedit /set hypervisorlaunchtype auto
→ Reboot

# Slow file access in /mnt/c
→ Store projects inside Linux FS: ~/projects, not /mnt/c/...
→ Use VS Code Remote WSL for editing

# sudo fails / password rejected
→ Reset password:
  # In PowerShell:
  wsl -d Ubuntu-24.04 -u root
  # Inside WSL:
  passwd kirill

# Port conflicts with Windows
→ Check reserved ports:
  netsh interface ipv4 show excludedportrange protocol=tcp
→ Or change port in your app
```

---

## Backup and migration

```powershell
# Export distro
wsl --export Ubuntu-24.04 D:\wsl-backups\ubuntu-$(Get-Date -Format 'yyyy-MM-dd').tar

# Import on another PC
wsl --import Ubuntu-24.04 D:\WSL\Ubuntu D:\wsl-backups\ubuntu-2026-03-16.tar --version 2

# Set default user after import
# Create: /etc/wsl.conf in imported distro
[user]
default=kirill
```

---

## Links

- 🌐 [Official WSL docs](https://learn.microsoft.com/windows/wsl/)
- 📦 [Distros in Microsoft Store](https://aka.ms/wslstore)
- ⚙️ [.wslconfig settings](https://learn.microsoft.com/windows/wsl/wsl-config)
- 🐳 [Docker + WSL2](https://docs.docker.com/desktop/wsl/)
