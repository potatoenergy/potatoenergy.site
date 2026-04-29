---
author: ["Potato Energy Team", "ponfertato"]
categories: ["home-assistant", "docker", "bluetooth", "guide"]
date: "2026-04-29T12:45:00+03:00"
description: "Home Assistant в Docker: настройка Bluetooth для обнаружения устройств. Проброс D-Bus, пассивное сканирование, интеграция с Meshtastic."
draft: false
series: ["Home Assistant"]
slug: "bluetooth-docker"
tags: ["home-assistant", "docker", "bluetooth", "meshtastic", "ble"]
title: "Home Assistant + Docker: Bluetooth для обнаружения устройств"
---

Home Assistant в Docker не видит Bluetooth-устройства, даже если на хосте всё работает.

**Причины**:

- ❌ Контейнер не имеет прямого доступа к `/dev/hci0`
- ❌ BlueZ внутри контейнера конфликтует с демоном на хосте
- ❌ Пассивное сканирование BLE требует флага `--experimental` в BlueZ

**Решение**: не запускать Bluetooth-стек внутри контейнера, а пробросить D-Bus с хоста.

---

## ✅ Настройка Docker Compose

### Минимальная конфигурация

```yaml
services:
  home-assistant:
    container_name: home-assistant
    image: ghcr.io/home-assistant/home-assistant:stable
    volumes:
      - config:/config
      - /run/dbus:/run/dbus:ro # ← Критично для Bluetooth
    cap_add:
      - NET_ADMIN
      - NET_RAW
      - SYS_ADMIN
    restart: unless-stopped
    networks:
      - traefik
      - prometheus

volumes:
  config:
    driver: local

networks:
  traefik:
    external: true
    name: traefik
  prometheus:
    external: true
    name: prometheus
```

> ⚠️ **Не добавляйте `devices: - /dev/hci0:/dev/hci0`** - это не нужно при пробросе D-Bus и может вызвать конфликт.

---

## 🔧 Настройка BlueZ на хосте

### Включение экспериментального режима (для пассивного сканирования)

Home Assistant использует пассивное сканирование BLE, которое требует запуска `bluetoothd` с флагом `--experimental`.

```bash
# 1. Создайте оверлей для systemd
sudo systemctl edit bluetooth

# 2. Вставьте:
[Service]
ExecStart=
ExecStart=/usr/lib/bluetooth/bluetoothd --experimental

# 3. Примените и перезапустите
sudo systemctl daemon-reload
sudo systemctl restart bluetooth

# 4. Проверьте, что флаг применён
ps aux | grep bluetoothd
# Ожидаемо: /usr/lib/bluetooth/bluetoothd --experimental
```

> 💡 **Почему это нужно**: пассивное сканирование не отправляет активные запросы, что экономит батарею устройств. Но эта функция в BlueZ помечена как «экспериментальная».

---

## 🔗 Сопряжение устройств (на хосте!)

Управляйте Bluetooth **только на хосте**, не внутри контейнера.

```bash
# 1. Включите адаптер
sudo bluetoothctl power on

# 2. Запустите сканирование
sudo bluetoothctl scan on

# 3. Когда устройство появится (например, Meshtastic_XXXX):
sudo bluetoothctl pair AA:BB:CC:DD:EE:FF
sudo bluetoothctl trust AA:BB:CC:DD:EE:FF
sudo bluetoothctl connect AA:BB:CC:DD:EE:FF
```

> ✅ После сопряжения на хосте, Home Assistant увидит устройство через проброшенный D-Bus.

---

## ⚙️ Интеграция в Home Assistant

### Установка интеграции

1. Установите [Meshtastic через HACS](https://github.com/meshtastic/meshtastic-ha) (или другую нужную интеграцию)
2. Перезапустите HA: `docker compose restart home-assistant`
3. Настройки → Интеграции → Добавить интеграцию → [Название]
4. Устройство должно появиться в списке обнаруженных (по имени или адресу)

### Если устройство не обнаруживается

```bash
# Проверьте на хосте:
- Устройство спарено? `bluetoothctl paired-devices`
- Адаптер не заблокирован? `rfkill list`
- BlueZ запущен с --experimental? `ps aux | grep bluetoothd`

# В Home Assistant:
- Перезагрузите интеграцию: Настройки → Система → Перезагрузить
- Проверьте логи: Настройки → Система → Логи → поиск "bluetooth"
```

---

## ⚠️ Частые проблемы

| Симптом                                   | Причина                              | Решение                                            |
| ----------------------------------------- | ------------------------------------ | -------------------------------------------------- |
| `Unable to open mgmt_socket` в контейнере | Конфликт демонов                     | Не запускайте `bluetoothctl` внутри контейнера     |
| Устройство не появляется в скане          | Не включено сопряжение на устройстве | В приложении устройства: включить режим сопряжения |
| `Error.Busy` при включении питания        | `hciattach` держит устройство        | Убрать флаг `-n` в сервисе инициализации           |
| Пассивное сканирование не работает        | BlueZ без `--experimental`           | Добавить флаг в `ExecStart` и перезапустить        |

---

## Ссылки

- 🏠 [Home Assistant Bluetooth Docs](https://www.home-assistant.io/integrations/bluetooth/)
- 📘 [BlueZ Experimental Features](https://www.bluez.org/experimental/)
- 🐙 [Meshtastic HA Integration](https://github.com/meshtastic/meshtastic-ha)
- 🐳 [Docker Bind Mounts](https://docs.docker.com/storage/bind-mounts/)
