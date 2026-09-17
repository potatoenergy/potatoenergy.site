---
author: ["Potato Energy Team", "ponfertato"]
categories: ["hardware", "orangepi", "bluetooth", "guide"]
date: "2026-04-29T12:40:00+03:00"
description: "Настройка Bluetooth на Orange Pi 3B: чип Spreadtrum UWE5622, hciattach, BlueZ. Решение проблемы 'No default controller' и 'Error.Busy'."
draft: false
series: ["Hardware"]
slug: "orangepi3b-bluetooth"
tags: ["orangepi", "bluetooth", "spreadtrum", "bluez", "hciattach", "linux"]
title: "Orange Pi 3B: Включаем Bluetooth (Spreadtrum UWE5622)"
aliases:
  - /blog/linux/orangepi/orangepi3b-bluetooth/
---

На Orange Pi 3B встроенный Bluetooth-чип **Spreadtrum UWE5622** подключён через UART (`/dev/ttyBT0`). В отличие от USB-адаптеров, он требует:

1. Загрузки прошивки и калибровочных данных перед инициализацией
2. Запуска `hciattach_opi` с правильными флагами
3. Корректного порядка запуска: сначала инициализация чипа, потом демон BlueZ

**Симптомы**:

- `bluetoothctl scan on` → `No default controller available`
- `btmgmt info` → `Index list with 0 items`
- `hciconfig -a` показывает `hci0`, но `bluetoothctl` его не видит
- Ошибка `org.bluez.Error.Busy` при попытке включить питание

**Причина**: сервис `orangepi3b-sprd-bluetooth.service` запускает `hciattach_opi` с флагом `-n` (no-detach), который удерживает устройство, не давая BlueZ зарегистрировать контроллер.

---

## ✅ Решение: два шага

### Шаг 1: Установка недостающих утилит

```bash
sudo apt update
sudo apt install -y rfkill bluez
```

> `rfkill` нужен для разблокировки адаптера, `bluez` - сам стек Bluetooth.

### Шаг 2: Исправление сервиса инициализации

#### Вариант А: Ручное исправление

```bash
# 1. Остановить всё
sudo systemctl stop orangepi3b-sprd-bluetooth
sudo systemctl stop bluetooth
sudo killall -9 hciattach_opi 2>/dev/null
sleep 2

# 2. Загрузить драйверы и разблокировать адаптер
sudo modprobe -a sprdbt_tty sprdwl_ng
sudo rfkill unblock all

# 3. Запустить инициализацию без флага -n (hciattach отцепится автоматически)
sudo /usr/bin/hciattach_opi -s 1500000 /dev/ttyBT0 sprd
# ← Процесс завершится через 2-3 секунды, если чип ответил

# 4. Запустить BlueZ - он подхватит готовый hci0
sudo systemctl start bluetooth
sleep 2

# 5. Проверка
btmgmt info
# Ожидаемо: Index list with 1 item, hci0: Primary controller

sudo bluetoothctl power on
sudo bluetoothctl scan on
# Ожидаемо: Changing power on succeeded, устройство появится в скане
```

#### Вариант Б: Постоянное исправление (через systemd)

```bash
# 1. Открыть сервис для редактирования
sudo systemctl edit orangepi3b-sprd-bluetooth

# 2. Переопределить ExecStart (убрать -n):
[Service]
ExecStart=
ExecStart=/usr/bin/hciattach_opi -s 1500000 /dev/ttyBT0 sprd

# 3. Применить и перезапустить
sudo systemctl daemon-reload
sudo systemctl restart orangepi3b-sprd-bluetooth
sudo systemctl restart bluetooth

# 4. Проверка
btmgmt info
sudo bluetoothctl power on
sudo bluetoothctl scan on
```

> 💡 **Почему `-n` ломает работу**: флаг `no-detach` заставляет `hciattach` оставаться в фоне и удерживать UART. BlueZ не может зарегистрировать контроллер, потому что устройство уже занято.

---

## 🔧 Если не работает: диагностика

```bash
# Проверить, видит ли ядро чип
dmesg | grep -iE "ttyBT|sprd|uwe5622"

# Проверить, загружены ли драйверы
lsmod | grep -iE "sprd|bt"

# Проверить блокировки
rfkill list
# Если Soft blocked: yes → sudo rfkill unblock bluetooth

# Проверить, запущен ли bluetoothd с нужными флагами
ps aux | grep bluetoothd
# Для пассивного сканирования (нужно для HA) требуется: --experimental

# Проверить права на устройство
ls -l /dev/hci0 /dev/ttyBT0
# Должно быть: root:bluetooth, права 660
```

---

## 🐳 Для Docker: проброс в контейнер

Если запускается Home Assistant или другое приложение в Docker:

```yaml
services:
  home-assistant:
    volumes:
      - /run/dbus:/run/dbus:ro # ← Обязательно для доступа к Bluetooth
    # Не запускать bluetoothctl внутри контейнера!
    # Управлять сопряжением на хосте, приложение подхватит через D-Bus
```

> ⚠️ **Не использовать `network_mode: host` только ради Bluetooth** - это ломает сетевую изоляцию. Проброса `/run/dbus` достаточно.

---

## Ссылки

- [Orange Pi 3B Wiki](https://wiki.orangepi.org/)
- [BlueZ Documentation](https://www.bluez.org/)
- [hciattach Source](https://git.kernel.org/pub/scm/bluetooth/bluez.git/tree/tools/hciattach.c)
