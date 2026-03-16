---
author: ["Potato Energy Team", "ponfertato"]
categories: ["linux", "arm", "recovery", "guide"]
date: "2026-03-17T01:00:00+03:00"
description: "Data recovery and migration on OrangePI: working with eMMC, USB drives, chroot, and rsync."
draft: false
series: ["Linux"]
slug: "storage-recovery"
tags: ["orangepi", "arm", "recovery", "rsync", "chroot", "storage"]
title: "OrangePI: Data Recovery & Migration"
---

OrangePI and other ARM single-board computers often use eMMC or SD cards with limited write endurance. Over time, storage fills up, degrades, or fails. This guide describes a method for data recovery and migration to external storage (USB SSD/HDD) via a chroot environment.

> 💡 Method is universal: works for OrangePI, Raspberry Pi, NanoPi, and other ARM systems.

---

## 📦 Preparation

### Requirements

| Component  | Requirements                           |
| ---------- | -------------------------------------- |
| USB drive  | SSD/HDD, capacity ≥ eMMC data          |
| Live image | Any Linux with ARM support (optional)  |
| Access     | root or sudo, physical access to board |
| Network    | Ethernet or WiFi (for remote access)   |

### Check connected storage

```bash
# Show all MMC devices (eMMC, SD card)
ls /dev/mmc*

# Show all block devices
lsblk

# Show partitions and mount points
df -h
```

**Expected output**:

- `/dev/mmcblk0` - built-in eMMC or SD card
- `/dev/sda`, `/dev/sdb` - connected USB drives

---

## 🔧 Mounting partitions

### Step 1: Mount eMMC root filesystem

```bash
# Create mount point
mkdir -p /mnt

# Mount root partition (usually mmcblk0p1 or p2)
mount /dev/mmcblk0p2 /mnt
```

**Why**:

- Direct access to filesystem for diagnostics
- Ability to boot from USB but work with eMMC data

### Step 2: Connect external storage

```bash
# Create USB mount point
mkdir -p /mnt/usb

# Mount USB partition
mount /dev/sda1 /mnt/usb
```

**Why USB**:

- eMMC has limited write cycles (~10K cycles)
- USB SSD is more reliable for long-term storage
- Easier to replace when degraded

### Step 3: Verify accessibility

```bash
# Confirm partitions are mounted
df -h /mnt
df -h /mnt/usb

# Check access permissions
ls -la /mnt/
ls -la /mnt/usb/
```

---

## 🚀 Working via chroot

### Enter chroot environment

```bash
# Change root to mounted system
chroot /mnt /bin/bash
```

**What chroot provides**:

- Work inside original system, not live image
- Access to installed packages, configs, services
- Run commands as the target system

### Verify inside chroot

```bash
# Confirm we're in the right system
hostname
cat /etc/os-release

# Check mounts inside chroot
df -h
```

> ⚠️ If `/proc`, `/sys`, `/dev` needed - mount before chroot:
>
> ```bash
> mount -t proc proc /mnt/proc
> mount -t sysfs sys /mnt/sys
> mount --bind /dev /mnt/dev
> ```

---

## 📊 Analyze disk usage

### Check usage by directory

```bash
# Home directory size
du -h /home

# Project/application size
du -h /opt/project

# Overall partition stats
df -h /
```

**Why `du -h`**:

- `-h` - human-readable format (K, M, G)
- Shows actual disk usage, not file sizes
- Helps find "space hogs"

### Find large files

```bash
# Find files > 100MB
find / -type f -size +100M -exec ls -lh {} \;

# Top 10 largest directories
du -h / | sort -rh | head -10
```

---

## 📋 Data migration via rsync

### Basic sync

```bash
# Copy home directory and projects to USB
rsync -av /home /opt/project /mnt/usb/
```

**rsync options**:

- `-a` - archive mode (preserves permissions, links, timestamps)
- `-v` - verbose (shows copy progress)

### Advanced sync

```bash
# With progress and deletion of extra files on target
rsync -av --progress --delete /home /opt/project /mnt/usb/

# With compression for slow connections
rsync -avz --progress /home /opt/project /mnt/usb/
```

**Why rsync, not cp**:

- Copies only changed files on re-run
- Preserves all metadata (owner, permissions, timestamps)
- Shows progress and speed
- Can be interrupted and resumed

### Verify after copy

```bash
# Compare source and destination sizes
du -sh /home /opt/project
du -sh /mnt/usb/home /mnt/usb/project

# Check checksums (optional)
md5sum /home/user/.bashrc
md5sum /mnt/usb/home/user/.bashrc
```

---

## 🔄 Boot from USB (optional)

### Change boot configuration

```bash
# For OrangePI with U-Boot
# Edit /boot/boot.cmd or /boot/uEnv.txt
# Specify USB as root partition

# Example for uEnv.txt:
setenv bootargs 'console=ttyS0,115200 root=/dev/sda1 rootwait'
```

### Or via fstab

```bash
# Mount USB as root on boot
# /etc/fstab
/dev/sda1  /  ext4  defaults,noatime  0  1
```

**Why boot from USB**:

- Offload eMMC from writes (system logs, cache)
- Extend built-in storage lifespan
- Easy replacement/upgrade without reflashing

---

## 🛡 Recovery after failure

### If system won't boot

```bash
# Boot from live image (SD card or network)
# Mount eMMC and USB as described above
# Use rsync to recover data
```

### If eMMC fully degraded

```bash
# Move entire system to USB
rsync -avx / /mnt/usb/

# Reinstall bootloader on USB
# For U-Boot:
dd if=/usr/lib/u-boot/orangepi_pc_plus/u-boot-sunxi-with-spl.bin of=/dev/sda bs=1024 seek=8
```

> ⚠️ Bootloader operations require exact board model matching!

---

## 🔧 Diagnostic commands

```bash
# Check eMMC health (if supported)
smartctl -a /dev/mmcblk0

# Check write errors in logs
dmesg | grep -iE 'error|fail|mmc'

# Get USB drive model and speed
lsusb -t
hdparm -I /dev/sda

# Test read/write speed
dd if=/dev/zero of=/mnt/usb/test bs=1M count=1024 conv=fdatasync
dd if=/mnt/usb/test of=/dev/null bs=1M
```

---

## ⚠️ Common issues

```bash
# USB not detected
→ Check power: some SSDs need external power supply
→ Try different USB 3.0 port
→ Check dmesg | tail after connection

# Mount error
→ Check filesystem: fsck /dev/sda1
→ Ensure partition not busy: umount /dev/sda1

# rsync interrupted
→ Use --partial for resume:
  rsync -av --partial /source /dest
→ Check power stability

# chroot not working
→ Mount pseudo-FS before chroot:
  mount -t proc proc /mnt/proc
  mount --bind /dev /mnt/dev
→ Check architecture: uname -m
```

---

## Links

- 🌐 [OrangePI Official Wiki](http://www.orangepi.org/)
- 📦 [Armbian Documentation](https://docs.armbian.com/)
- 🔧 [rsync Man Page](https://man7.org/linux/man-pages/man1/rsync.1.html)
- 💾 [eMMC vs SSD Lifespan](https://www.ssdlife.com/)
