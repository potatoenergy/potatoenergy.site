---
author: ["Potato Energy Team", "ponfertato"]
categories: ["linux", "nixos", "recovery", "guide"]
date: "2026-04-26T17:00:00+03:00"
description: "Recovering a corrupted file system and Nix Store in NixOS: bypassing read-only traps and verifying store integrity."
draft: false
series: ["Nix/NixOS"]
slug: "fs-recovery"
tags: ["nixos", "ext4", "btrfs", "e2fsck", "recovery", "nix-store"]
title: "NixOS: FS and Nix Store Recovery on Critical Failures"
---

The system fails to boot, dropping into `emergency mode`, or throws this error during operation:

```text
Structure needs cleaning
```

This indicates critical metadata corruption (superblock, inode table, or journal). The kernel forcibly mounts the root (`/`) as **Read-Only** to protect the data.

### ⚠️ The "In-System" Recovery Trap

Attempting to fix the root FS from within the running system is a dead end:

1. `mount -o remount,ro /` returns `target is busy` because background processes hold files open.
2. Mass-killing processes (`killall5`, `fuser`) often causes session hangs or respawns.
3. Using `umount -l /` (lazy unmount) detaches the root, but **instantly makes all binaries unavailable** (`sudo`, `e2fsck`, `mount`), as they reside in `/nix/store`, which is now unmounted.

> 💡 **Conclusion:** Fixing the root FS from within itself is like changing an engine while driving. The only reliable path is external intervention.

---

## 🛠 The Solution: External Recovery

For safe repair, the disk must be connected to another system (a second PC or LiveUSB) where it is not the root drive.

### Step 1: Forced Check and Repair

Connect the disk to another PC or boot from a Live medium (GParted Live, Ubuntu, Arch).

1. **Identify the corrupted partition:**

   ```bash
   lsblk -f
   ```

   Find the partition with the target UUID (e.g., `/dev/sda2` or `/dev/nvme0n1p2`).

2. **Run aggressive check (for ext4):**

   ```bash
   sudo e2fsck -y -f /dev/sdXn
   ```

   - `-y`: automatically answer "yes" to all fixes.
   - `-f`: force check even if the FS is marked "clean".

3. **If the check fails or cannot find the superblock:**
   Restore from a backup superblock (standard backups: `32768`, `98304`, `163840`):

   ```bash
   sudo e2fsck -b 32768 -y /dev/sdXn
   ```

4. **For Btrfs:**
   ```bash
   sudo btrfs check --repair /dev/sdXn
   ```

---

## 🔧 Step 2: Return to System and Nix Store Verification

After a successful `e2fsck`, the physical disk integrity is restored. However, the logical structure of `/nix/store` (especially the `.links` hardlink table) might have been corrupted during the crash.

Boot into your NixOS **without** the `fsck.mode=skip` parameter.

### 1. Verify and Repair Store Hashes

This command checks all store paths and automatically redownloads/rebuilds corrupted ones from the cache:

```bash
sudo nix-store --verify --check-contents --repair
```

> ⚠️ If this throws a `Bad message` error on files in `/nix/store/.links/`, the hardlink table is destroyed. Proceed to step 2.

### 2. Force Store Rebuild (if Step 1 fails)

If `.links` are corrupted, force Nix to recreate them from scratch:

```bash
# Delete broken hardlinks (use find to avoid "Argument list too long")
sudo find /nix/store/.links -mindepth 1 -delete

# Recreate optimization
sudo nix-store --optimise
```

### 3. Restore System Symlinks

Recreate `/run/current-system` and update the bootloader using only valid paths:

```bash
sudo nixos-rebuild boot --repair
```

### 4. Clean Up Garbage

Remove old generations and broken links to free up space and solidify the fix:

```bash
sudo nix-collect-garbage -d
```

---

## ⚠️ Common Issues and Solutions

| Symptom                                 | Cause                                               | Solution                                                                   |
| --------------------------------------- | --------------------------------------------------- | -------------------------------------------------------------------------- |
| `target is busy` on `remount,ro`        | Processes hold files in `/`                         | Don't fight it. Use LiveUSB/another PC.                                    |
| `command not found` after `umount -l /` | Root is unmounted, PATH is lost                     | This is expected. Hard reboot (`reboot -f`) and repair externally.         |
| `Bad message` in `/nix/store/.links/`   | ext4 hardlink table corrupted                       | Run `find /nix/store/.links -mindepth 1 -delete` + `nix-store --optimise`. |
| System drops to RO again on boot        | `e2fsck` wasn't run with `-f` or damage is critical | Repeat Step 1 with `-b 32768` (superblock recovery).                       |

---

## Links

- 📘 [NixOS Manual: Repairing the Nix Store](https://nixos.org/manual/nixos/stable/#sec-nix-store-repair)
- 🛠️ [e2fsck Documentation](https://man7.org/linux/man-pages/man8/e2fsck.8.html)
- 🥔 [Potato Energy: OrangePI Data Recovery](/blog/linux/orangepi/storage-recovery-en/)
