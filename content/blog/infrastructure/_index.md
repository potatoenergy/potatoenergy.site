---
description: "Серверная часть проекта: контейнеры, сети, ARM-платформы"
title: "Инфраструктура"
---

### Серверная часть проекта

Развёртывание, обслуживание и восстановление серверных систем. Материалы охватывают контейнеризацию, настройку сетевого оборудования и работу с ARM-платформами.

**Подразделы**

**Docker** - управление контейнерами и данными

- [Бэкап и миграция volumes](docker/volumes-management/) - универсальные методы работы с томами
- [Автоматическое выключение по расписанию](docker/scheduled-shutdown/) - экономия ресурсов на домашних серверах
- [Очистка и оптимизация дисков WSL](docker/wsl-cleanup/) - сжатие VHDX в Windows
- [Миграция PostgreSQL между версиями](docker/postgres-upgrade/) - безопасное обновление без потери данных

**OpenWrt** - маршрутизация и сетевая безопасность

- [Продвинутая настройка роутера](openwrt/setup/) - DNS-over-HTTPS, WireGuard, firewall, Prometheus

**ARM-серверы** - одноплатные компьютеры

- [Восстановление и миграция данных на OrangePi](arm-servers/storage-recovery/) - eMMC, USB, chroot, rsync
- [Включение Bluetooth на Orange Pi 3B](arm-servers/orangepi3b-bluetooth/) - Spreadtrum UWE5622, hciattach, BlueZ
