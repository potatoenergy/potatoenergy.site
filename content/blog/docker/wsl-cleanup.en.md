---
author: ["Potato Energy Team", "ponfertato"]
categories: ["docker", "windows", "wsl", "guide"]
date: "2026-03-17T09:30:00+03:00"
description: "Clean up Docker Desktop on WSL2: free disk space, optimize VHDX, automate."
draft: false
series: ["Docker"]
slug: "wsl-cleanup"
tags: ["docker", "wsl", "windows", "powershell", "cleanup"]
title: "Docker WSL: Disk Cleanup & Optimization"
---

Docker Desktop on Windows uses WSL2 with dynamic VHDX files. They **grow** when adding images/containers but **don't shrink automatically** when deleting them. Result: `C:` drive fills up, even though `docker system df` shows free space.

> 💡 Solution: manual VHDX optimization via PowerShell + Docker utility.

---

## 📦 Cleanup script

### Create file

```powershell
# File: $HOME\Scripts\docker-clear-wsl.ps1
$script = @'
$LOCAL = "$env:LOCALAPPDATA\Docker\wsl"
$VHD1 = Join-Path $LOCAL "disk\docker_data.vhdx"
$VHD2 = Join-Path $LOCAL "main\ext4.vhdx"

# 1. Docker cleanup
docker system prune -f

# 2. Reclaim space via official tool
docker run --rm --privileged --pid=host docker/desktop-reclaim-space
docker rmi docker/desktop-reclaim-space -f

# 3. Stop Docker Desktop
Get-Process -Name "Docker Desktop","com.docker.backend","com.docker.build" `
  -ErrorAction SilentlyContinue | Stop-Process -Force -ErrorAction SilentlyContinue

# 4. Shutdown WSL
wsl --shutdown

# 5. Optimize VHDX files (compact)
if (Test-Path $VHD1) { Optimize-VHD -Path $VHD1 -Mode Full }
if (Test-Path $VHD2) { Optimize-VHD -Path $VHD2 -Mode Full }

# 6. Restart Docker Desktop
Start-Sleep -Seconds 2
Start-Process -FilePath "$env:ProgramFiles\Docker\Docker\Docker Desktop.exe" `
  -ErrorAction SilentlyContinue
'@

$script | Out-File -FilePath "$HOME\Scripts\docker-clear-wsl.ps1" -Encoding UTF8
```

### Run

```powershell
# As Administrator (required for Optimize-VHD)
powershell.exe -ExecutionPolicy Bypass -File "$HOME\Scripts\docker-clear-wsl.ps1"
```

---

## 🔍 How it works

| Step | Command                        | What it does                                               |
| ---- | ------------------------------ | ---------------------------------------------------------- |
| 1    | `docker system prune -f`       | Removes stopped containers, unused images, build cache     |
| 2    | `docker/desktop-reclaim-space` | Official Docker tool to reclaim space in WSL2              |
| 3    | `Stop-Process`                 | Gracefully stops Docker Desktop (otherwise VHDX is locked) |
| 4    | `wsl --shutdown`               | Fully shuts down WSL, freeing files for optimization       |
| 5    | `Optimize-VHD -Mode Full`      | Compacts VHDX files, returning space to host               |
| 6    | `Start-Process`                | Restarts Docker Desktop                                    |

**Why this way**:

- Without stopping processes, `Optimize-VHD` won't work (file locked)
- `docker/desktop-reclaim-space` works inside WSL, removing "ghost" data
- `Mode Full` - maximum compression (slower, but more effective)

---

## ⚙️ Automation

### Via Task Scheduler (GUI)

1. Open **Task Scheduler** → Create Basic Task
2. Trigger: Weekly (e.g., Sunday, 03:00)
3. Action: Start a program
   - Program: `powershell.exe`
   - Arguments: `-ExecutionPolicy Bypass -File "C:\Users\ponfertato\Scripts\docker-clear-wsl.ps1"`
4. Settings: ✅ Run with highest privileges

### Via PowerShell (scripted)

```powershell
# Create task as Administrator
$action = New-ScheduledTaskAction -Execute "powershell.exe" `
  -Argument "-ExecutionPolicy Bypass -File `"$HOME\Scripts\docker-clear-wsl.ps1`""

$trigger = New-ScheduledTaskTrigger -Weekly -DaysOfWeek Sunday -At 3am

$principal = New-ScheduledTaskPrincipal -UserId "SYSTEM" -RunLevel Highest

Register-ScheduledTask -TaskName "Docker WSL Cleanup" `
  -Action $action -Trigger $trigger -Principal $principal
```

### Verify and run manually

```powershell
# Run task
Start-ScheduledTask -TaskName "Docker WSL Cleanup"

# Check status
Get-ScheduledTask -TaskName "Docker WSL Cleanup" | Get-ScheduledTaskInfo

# View history
Get-WinEvent -FilterHashtable @{LogName='Microsoft-Windows-TaskScheduler/Operational'; TaskName='Docker WSL Cleanup'} -MaxEvents 10
```

---

## 📊 Monitor results

### Before and after

```powershell
# VHDX size before cleanup
Get-ChildItem "$env:LOCALAPPDATA\Docker\wsl\disk\docker_data.vhdx" | Select Name, @{N="SizeGB";E={[math]::Round($_.Length/1GB,2)}}

# After cleanup - compare value
```

### Docker stats

```powershell
# Summary
docker system df

# Detailed
docker system df -v
```

**Expected result**:

- `docker system df` shows less data
- `.vhdx` file size on disk decreases (sometimes 2-5x)

---

## ⚠️ Safety and notes

### Requirements

| Requirement          | Why it matters                              |
| -------------------- | ------------------------------------------- |
| Run as Administrator | `Optimize-VHD` requires elevated privileges |
| Close WSL-using apps | Otherwise `wsl --shutdown` will fail        |
| Backup critical data | In case of failure (unlikely, but possible) |

### If something goes wrong

```powershell
# Docker won't start after cleanup
→ Launch manually: "$env:ProgramFiles\Docker\Docker\Docker Desktop.exe"
→ Check logs: Get-Content "$env:LOCALAPPDATA\Docker\log.txt" -Tail 50

# VHDX didn't compact
→ Ensure Docker and WSL are fully stopped: wsl --list --verbose
→ Try manually: Optimize-VHD -Path "C:\path\to\docker_data.vhdx" -Mode Full

# "Access denied" error
→ Run PowerShell as Administrator
→ Check antivirus: may block VHDX access
```

### Exclude from cleanup (optional)

To preserve specific images:

```powershell
# Before prune, tag images
docker tag my-important-image my-important-image:keep
docker system prune -f --filter "label!=keep"
```

---

## 🗂 Pre-run checklist

- [ ] Closed all apps using Docker/WSL
- [ ] Launched PowerShell as Administrator
- [ ] Checked free space on `C:` (need ~10% of VHDX size for compaction)
- [ ] Ensured critical data is saved or tagged
- [ ] Tested on one VHDX before full cleanup

---

## Links

- 🐳 [Docker Desktop WSL2 Backend](https://docs.docker.com/desktop/wsl/)
- 🔧 [Optimize-VHD Docs](https://learn.microsoft.com/powershell/module/hyper-v/optimize-vhd)
- 🗑️ [docker system prune](https://docs.docker.com/engine/reference/commandline/system_prune/)
- 🔄 [Reclaim disk space in WSL2](https://docs.docker.com/desktop/troubleshoot/topics/#disk-space)
