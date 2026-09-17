---
description: "Server-side of the project: containers, networking, ARM platforms"
title: "Infrastructure"
---

### Server-side of the project

Deployment, maintenance, and recovery of server systems. Materials cover containerization, network equipment configuration, and ARM platforms.

**Subsections**

**Docker** - container and data management

- [Volume Backup & Migration](docker/volumes-management/) - universal volume management methods
- [Scheduled Automatic Shutdown](docker/scheduled-shutdown/) - resource saving on home servers
- [WSL Disk Cleanup & Optimization](docker/wsl-cleanup/) - VHDX compaction on Windows
- [PostgreSQL Major Version Migration](docker/postgres-upgrade/) - safe upgrade without data loss

**OpenWrt** - routing and network security

- [Advanced Router Configuration](openwrt/setup/) - DNS-over-HTTPS, WireGuard, firewall, Prometheus

**ARM Servers** - single-board computers

- [OrangePI Data Recovery & Migration](arm-servers/storage-recovery/) - eMMC, USB, chroot, rsync
- [Enabling Bluetooth on Orange Pi 3B](arm-servers/orangepi3b-bluetooth/) - Spreadtrum UWE5622, hciattach, BlueZ
