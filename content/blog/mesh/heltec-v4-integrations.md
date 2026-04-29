---
author: ["Potato Energy Team", "ponfertato"]
categories: ["mesh", "home-assistant", "mqtt", "telegram", "guide"]
date: "2026-04-29T113:15:00+03:00"
description: "Meshtastic + Home Assistant + ONEmesh: интеграция через MQTT, уведомления в Telegram, мониторинг нод. Часть 3: Интеграции."
draft: false
series: ["Mesh Networks"]
slug: "integrations"
tags: ["meshtastic", "home-assistant", "mqtt", "telegram", "onemesh", "yaml"]
title: "Heltec V4: Интеграции. Home Assistant, Telegram, мониторинг. Часть 3"
---

> 📌 **Это третья часть цикла**. [Часть 1: Теория]({{< ref "/blog/mesh/heltec-v4-mesh-intro.md" >}}), [Часть 2: Практика]({{< ref "/blog/mesh/heltec-v4-practice.md >}}).

---

## 🔗 Что будем интегрировать

| Компонент          | Назначение                                                              | Сложность  |
| ------------------ | ----------------------------------------------------------------------- | ---------- |
| **ONEmesh MQTT**   | Мост между локальной сетью и глобальной картой + уведомления в Telegram | 🟢 Низкая  |
| **Home Assistant** | Сенсоры нод, device_tracker, автоматизации на сообщения из сети         | 🟡 Средняя |
| **Уведомления**    | Алерты о новых нодах, потере связи, упоминаниях в чате                  | 🟢 Низкая  |

> 💡 Все примеры проверены на: HA 2026.4.2 (Docker) + Orange Pi 3B + встроенный MQTT-брокер.

---

## 🌐 Шаг 1: Настройка MQTT в Meshtastic для ONEmesh

### Параметры подключения

| Параметр              | Значение          | Примечание                                                      |
| --------------------- | ----------------- | --------------------------------------------------------------- |
| **MQTT Address**      | `mqtt.onemesh.ru` | Сервер сообщества                                               |
| **Username**          | `onemesh`         | По умолчанию                                                    |
| **Password**          | `onecat`          | Общий для всех                                                  |
| **Root topic**        | `msh/RU/XXX`      | `XXX` = код города ([список](https://map.onemesh.ru/mqtt.html)) |
| **TLS enabled**       | `Yes`             | Если не работает - попробуйте `No`                              |
| **JSON enabled**      | `Yes`             | Обязательно для Home Assistant                                  |
| **Uplink / Downlink** | `Yes` / `No`      | Для мониторинга достаточно только uplink                        |

> ⚠️ **Важно**: в прошивке 2.5+ появилась поддержка нового формата ключей шифрования (PKI). Если используете шифрование - убедитесь, что ключи синхронизированы между нодами.

### Как проверить работу

1. Включите `JSON enabled` в настройках MQTT
2. Подключитесь к брокеру через `mosquitto_sub` или MQTT Explorer:

```bash
mosquitto_sub -h mqtt.onemesh.ru -u onemesh -P onecat \
  -t 'msh/RU/MOW/json/#' -v
```

3. Вы увидите сообщения вида:

```json
{
  "id": 452664778,
  "from": 2130636288,
  "payload": {
    "text": "Привет из локальной сети!",
    "battery_level": 87,
    "voltage": 4.12
  },
  "type": "text",
  "timestamp": 1714234567
}
```

---

## 📬 Шаг 2: Уведомления в Telegram через ONEmesh Bot

ONEmesh автоматически пересылает сообщения из вашей городской темы в приватный чат с ботом.

### Примеры уведомлений

```
[26.04.2026 20:52] ONEmesh Bot: LF | qwrt (https://map.onemesh.ru/?node_id=483535916)  QWRT Pocket
20, cтaндapт. a вoт для дoмaшнeй нoды aнтeннa тpиaдa
SNR 4.75  RSSI -92  HopLimit 3
- данные от 44b4 (https://map.onemesh.ru/?node_id=49169588)
```

| Поле       | Что означает             | Зачем нужно                                 |
| ---------- | ------------------------ | ------------------------------------------- |
| `SNR`      | Signal-to-Noise Ratio    | Качество сигнала: чем выше, тем лучше       |
| `RSSI`     | Received Signal Strength | Мощность сигнала: ближе к 0 = лучше         |
| `HopLimit` | Оставшееся число прыжков | Показывает, насколько далеко ушло сообщение |

### Как настроить

1. Найдите бота [@ONEmeshBot](https://t.me/ONEmeshBot) в Telegram
2. Отправьте `/start` - бот покажет инструкции
3. Убедитесь, что ваша нода отправляет данные в MQTT с правильным `root topic`
4. Готово: уведомления будут приходить автоматически

> 💡 **Совет**: если уведомления не приходят - проверьте, что `OK to MQTT` включено в настройках канала и нода имеет интернет-соединение.

---

## 🏠 Шаг 3: Интеграция с Home Assistant

### Предварительные требования

- [ ] Встроенный MQTT-брокер в HA (`Settings → Devices & Services → MQTT`)
- [ ] Meshtastic-нода с включённым `JSON enabled` в MQTT
- [ ] Доступ к `configuration.yaml` и возможность создавать `mqtt.yaml`

### Шаг 3.1: Подключение MQTT в HA

```yaml
# configuration.yaml
mqtt: !include mqtt.yaml
```

```yaml
# mqtt.yaml
broker: core-mosquitto # или адрес вашего брокера
port: 1883
username: homeassistant
password: !secret mqtt_password
# Если используете внешний брокер (например, ONEmesh):
# broker: mqtt.onemesh.ru
# port: 8883
# username: onemesh
# password: onecat
# certificate: /config/certs/ca.crt  # для TLS
```

### Шаг 3.2: Создание сенсоров для ноды

```yaml
# mqtt.yaml - продолжение
sensor:
  # Батарея ноды
  - name: "Meshtastic Node Battery"
    unique_id: "meshtastic_node_battery"
    state_topic: "msh/RU/MOW/json/LongFast/!abcd1234"
    value_template: >-
      {% if value_json.from == 2130636288 and value_json.payload.battery_level is defined %}
        {{ value_json.payload.battery_level | int }}
      {% else %}
        {{ this.state }}
      {% endif %}
    unit_of_measurement: "%"
    device_class: battery

  # Напряжение
  - name: "Meshtastic Node Voltage"
    unique_id: "meshtastic_node_voltage"
    state_topic: "msh/RU/MOW/json/LongFast/!abcd1234"
    value_template: >-
      {% if value_json.from == 2130636288 and value_json.payload.voltage is defined %}
        {{ value_json.payload.voltage | float | round(2) }}
      {% else %}
        {{ this.state }}
      {% endif %}
    unit_of_measurement: "V"
    device_class: voltage

  # Канал утилизации (загрузка эфира)
  - name: "Meshtastic Channel Utilization"
    unique_id: "meshtastic_chutil"
    state_topic: "msh/RU/MOW/json/LongFast/!abcd1234"
    value_template: >-
      {% if value_json.from == 2130636288 and value_json.payload.channel_utilization is defined %}
        {{ value_json.payload.channel_utilization | float | round(1) }}
      {% else %}
        {{ this.state }}
      {% endif %}
    unit_of_measurement: "%"
```

> 🔑 **Важно**: замените `2130636288` на десятичный `from` вашей ноды (найдите в MQTT Explorer или через `meshtastic --info`).

### Шаг 3.3: Отслеживание местоположения (device_tracker)

```yaml
# automations.yaml
- alias: "Meshtastic: Update node location"
  trigger:
    platform: mqtt
    topic: "msh/RU/MOW/json/LongFast/!abcd1234"
    value_template: >-
      {% if value_json.from == 2130636288 and 
            value_json.payload.latitude_i is defined and 
            value_json.payload.longitude_i is defined %}on{% endif %}
  action:
    service: device_tracker.see
    data:
      dev_id: "meshtastic_node_1"
      gps:
        - "{{ (trigger.payload_json.payload.latitude_i | int * 1e-7) }}"
        - "{{ (trigger.payload_json.payload.longitude_i | int * 1e-7) }}"
      battery: "{{ states('sensor.meshtastic_node_battery') | default(0) }}"
```

> 💡 Meshtastic хранит координаты в формате `integer * 1e7`, поэтому умножаем на `1e-7` для получения реальных значений.

### Шаг 3.4: Уведомления о сообщениях из сети

```yaml
# automations.yaml
- alias: "Meshtastic: Notify on mention"
  trigger:
    platform: mqtt
    topic: "msh/RU/MOW/json/LongFast/!abcd1234"
    value_template: >-
      {% if value_json.type == "text" and 
            value_json.payload.text is defined and 
            "ponf" in value_json.payload.text.lower() %}on{% endif %}
  action:
    service: notify.mobile_app_your_phone
    data:
      title: "📡 Meshtastic"
      message: "Упоминание: {{ trigger.payload_json.payload.text }}"
      data:
        priority: high
```

---

## 🔐 Безопасность: шифрование и ключи

### Что изменилось в прошивке 2.5+

| Версия    | Тип ключей              | Особенности                                     |
| --------- | ----------------------- | ----------------------------------------------- |
| **< 2.5** | Единый `PSK` для канала | Простая настройка, но уязвимо при компрометации |
| **≥ 2.5** | `PKI` + `shared secret` | Отдельные ключи для шифрования и аутентификации |

### Как настроить шифрование

1. В приложении Meshtastic: **Channel → Primary → PSK**
   - Включите шифрование
   - Скопируйте ключ (он же `AQ==` для базового канала)
2. В настройках MQTT: **Encryption enabled → Yes**
3. Убедитесь, что все ноды в сети используют одинаковый ключ

> ⚠️ **Критично**: если ключи не совпадают - сообщения будут приходить, но не расшифровываться. Проверяйте через `MQTT Explorer`.

### Где хранить ключи

- ✅ В `secrets.yaml` Home Assistant (не коммитить в Git!)
- ✅ В менеджере паролей (Bitwarden, KeePassXC)
- ❌ Не в публичных репозиториях, не в логах, не в скриншотах

---

## ⚠️ Частые проблемы

| Симптом                       | Причина                            | Решение                                                 |
| ----------------------------- | ---------------------------------- | ------------------------------------------------------- |
| Сенсоры не обновляются        | Неверный `from` в `value_template` | Найдите десятичный ID ноды через `meshtastic --info`    |
| Нет сообщений в Telegram      | Неправильный `root topic`          | Проверьте код города: `msh/RU/MOW`, `msh/RU/SPB` и т.д. |
| Шифрование не работает        | Ключи не совпадают                 | Скопируйте `PSK` точно, без пробелов, в обе ноды        |
| `device_tracker` не двигается | Координаты в неверном формате      | Умножайте `latitude_i`/`longitude_i` на `1e-7`          |
| HA не подключается к брокеру  | Неверные учётные данные            | Проверьте `username`/`password` и порт (1883/8883)      |

---

## Ссылки

- 📡 [Meshtastic MQTT Docs](https://meshtastic.org/docs/configuration/module/mqtt/)
- 🏠 [HA Meshtastic Integration](https://meshtastic.org/docs/software/home-assistant/)
- 🗺️ [ONEmesh Map](https://map.onemesh.ru)
- 🔐 [Meshtastic Security Guide](https://meshtastic.org/docs/configuration/security/)
