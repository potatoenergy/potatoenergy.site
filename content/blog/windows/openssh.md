---
author: ["Potato Energy Team", "ponfertato"]
categories: ["ssh", "windows", "tutorial"]
date: "2026-03-16T15:25:00+03:00"
description: "Install and configure OpenSSH server on Windows 10/11 and Server 2019+. Fast, via PowerShell."
draft: false
series: ["SSH"]
slug: "openssh"
tags: ["windows", "ssh", "openssh", "powershell", "server"]
title: "OpenSSH on Windows: Server Setup"
---

OpenSSH is a tool for secure remote access via the SSH protocol. It encrypts all traffic, supports key-based authentication, and works on Windows, Linux, and macOS.

> 💡 After setup, you can connect to your Windows PC like a Linux server: `ssh user@192.168.1.100`

---

## Requirements

- **OS**: Windows 10 (1809+), Windows 11, Windows Server 2019/2022
- **Privileges**: Administrator
- **Network**: Access to port 22 (local or remote)

---

## Installation (3 ways)

### Option 1: PowerShell (recommended)

```powershell
# Run as Administrator
# Install OpenSSH server
Add-WindowsCapability -Online -Name OpenSSH.Server~~~~0.0.1.0

# Verify installation
Get-WindowsCapability -Online | Where-Object Name -like 'OpenSSH*'
```

### Option 2: DISM (alternative)

```powershell
dism /Online /Add-Capability /CapabilityName:OpenSSH.Server~~~~0.0.1.0
```

### Option 3: Via Settings (GUI)

1. Settings → Apps → Optional features
2. "Add a feature" → find "OpenSSH Server" → Install

---

## Configure the service

```powershell
# Run as Administrator

# Enable auto-start for sshd
Set-Service -Name sshd -StartupType Automatic

# Start the service
Start-Service sshd

# Check status
Get-Service sshd

# Verify port 22 is listening
netstat -ano | findstr :22
```

---

## Firewall

```powershell
# Check for OpenSSH rule
Get-NetFirewallRule -Name *OpenSSH-Server* | Select Name, Enabled

# If missing, create it
New-NetFirewallRule -Name sshd `
  -DisplayName 'OpenSSH Server' `
  -Enabled True `
  -Direction Inbound `
  -Protocol TCP `
  -LocalPort 22 `
  -Action Allow `
  -Profile Any
```

---

## Test connection

```powershell
# From the same PC
ssh localhost

# From another device on the network
ssh <your_username>@<Windows_IP>

# Example:
ssh kirill@192.168.1.100
```

> 💡 First connection will ask to confirm the host key fingerprint - type `yes`.

---

## Key-based authentication (recommended)

### On the client (where you connect from)

```bash
# Generate key pair (if you don't have one)
ssh-keygen -t ed25519 -C "kirill@potatoenergy.ru"

# Copy public key to server
# For Windows server - manually:
type $env:USERPROFILE\.ssh\id_ed25519.pub
# Copy the output
```

### On the server (Windows)

```powershell
# Create .ssh folder in user profile
mkdir $env:USERPROFILE\.ssh -Force

# Create/edit authorized_keys
notepad $env:USERPROFILE\.ssh\authorized_keys

# Paste the public key (single line), save

# Set correct permissions (REQUIRED)
$acl = Get-Acl $env:USERPROFILE\.ssh\authorized_keys
$acl.SetAccessRuleProtection($true, $false)
$rule = New-Object System.Security.AccessControl.FileSystemAccessRule(
    $env:USERNAME, "Read", "Allow")
$acl.AddAccessRule($rule)
Set-Acl $env:USERPROFILE\.ssh\authorized_keys $acl
```

### Disable password login (optional, for security)

```powershell
# Edit config
notepad C:\ProgramData\ssh\sshd_config

# Find and change:
# PasswordAuthentication no
# PubkeyAuthentication yes

# Restart service
Restart-Service sshd
```

---

## Config: useful settings

File: `C:\ProgramData\ssh\sshd_config`

```ini
# Allow only specific users
AllowUsers kirill admin

# Change port (if 22 is busy)
Port 2222

# Disable root login
PermitRootLogin no

# Inactivity timeout
ClientAliveInterval 300
ClientAliveCountMax 2

# Logging
LogLevel VERBOSE
```

After changes:

```powershell
Restart-Service sshd
```

---

## Troubleshooting

```powershell
# Service won't start
→ Check logs: Get-WinEvent -LogName "OpenSSH/Operational" -MaxEvents 10

# Port 22 not listening
→ Check firewall: Get-NetFirewallRule -Name sshd
→ Check service status: Get-Service sshd

# "Permission denied (publickey,password)"
→ Verify permissions on authorized_keys (owner read-only)
→ Ensure public key is pasted as one line, no line breaks

# Connected but no file access
→ Check user permissions on Windows folders
→ Try running terminal as Administrator on the client
```

---

## Links

- 🌐 [Win32-OpenSSH official repo](https://github.com/PowerShell/Win32-OpenSSH)
- 📘 [Microsoft docs](https://learn.microsoft.com/windows-server/administration/openssh/openssh_install_firstuse)
- 🔑 [sshd_config generator](https://www.ssh.com/academy/ssh/sshd_config)
