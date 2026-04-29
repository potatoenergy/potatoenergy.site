---
author: ["Potato Energy Team", "ponfertato"]
categories: ["smart-home", "home-assistant", "configuration", "guide"]
date: "2026-04-29T12:20:00+03:00"
description: "Точная настройка домашней зоны в Home Assistant: координаты, высота, радиус. Через UI и YAML."
draft: false
slug: "location-config"
tags: ["home-assistant", "configuration", "gps", "zone", "yaml"]
title: "Home Assistant: Точная настройка домашней локации"
---

Домашняя зона (`zone.home`) - основа для:

- ✅ Детекции присутствия (автоматизации «пришёл/ушёл»)
- ✅ Расчёта времени восхода/заката (освещение, шторы)
- ✅ Прогноза погоды (привязка к координатам)
- ✅ Геозон для устройств и пользователей

Неточные координаты = ложные срабатывания, неверный прогноз, сбои автоматизаций.

---

## 🎯 Проблема стандартного мастера

При первой настройке Home Assistant предлагает выбрать локацию на карте. Но:

- ❌ Нет ручного ввода координат в мастере
- ❌ Автоопределение по IP часто даёт погрешность 1–10 км
- ❌ Высота над уровнем моря не запрашивается

**Решение**: настроить точные координаты после установки, через UI или YAML.

---

## 🔧 Способ 1: Через UI (после установки)

### Шаг 1: Получить точные координаты

Используйте устройство с GPS (телефон с приложением Home Assistant):

1. Установите [официальное приложение](https://companion.home-assistant.io/)
2. Дайте разрешение на точное местоположение
3. Включите датчик `device_tracker.<device>` в настройках приложения
4. Откройте **Developer Tools → States** в веб-интерфейсе
5. Найдите сущность `device_tracker.<your_device>`
6. Скопируйте атрибуты: `latitude`, `longitude`, `altitude`

### Шаг 2: Обновить домашнюю зону

1. **Settings → Areas, labels & zones → Zones**
2. Откройте `home`
3. Вставьте координаты в поля `Latitude` / `Longitude`
4. Сохраните

### Шаг 3: Задать высоту

1. **Settings → System → General**
2. Заполните `Elevation above sea level` (в метрах)
3. Сохраните

> 💡 **Радиус зоны**: по умолчанию 100 м. Увеличьте до 200–500 м, если дом в частном секторе или есть погрешность GPS.

---

## ⚙️ Способ 2: Через YAML (для продвинутых)

Добавьте в `configuration.yaml`:

```yaml
homeassistant:
  name: Home
  latitude: 55.7558 # ← пример, замените на свои
  longitude: 37.6173 # ← пример, замените на свои
  elevation: 156 # высота в метрах
  radius: 200 # радиус зоны в метрах
  unit_system: metric
  time_zone: "Europe/Moscow"
  country: "RU"
  currency: "RUB"
```

**Важно**:

- После изменения `configuration.yaml` выполните **Check Configuration** и **Restart**
- Если координаты заданы в YAML, поля в UI станут недоступны для редактирования

---

## 🔄 Альтернатива: действие `homeassistant.set_location`

Для динамического обновления (например, из автоматизации):

```yaml
automation:
  - alias: "Update home location"
    trigger:
      platform: time_pattern
      minutes: "/30" # каждые 30 минут
    action:
      - action: homeassistant.set_location
        data:
          latitude: "{{ states('sensor.gps_latitude') }}"
          longitude: "{{ states('sensor.gps_longitude') }}"
          elevation: "{{ states('sensor.gps_altitude') }}"
```

> ⚠️ Используйте с осторожностью: частое обновление локации может нарушить работу автоматизаций присутствия.

---

## ⚠️ Частые проблемы

```yaml
# "Away" не меняется на "Home" при возвращении
→ Проверьте радиус зоны: увеличьте до 300–500 м
→ Убедитесь, что `device_tracker` обновляется (интервал, мин. расстояние)

# Неверное время восхода/заката
→ Проверьте `elevation` и `time_zone` в настройках
→ Перезагрузите интеграцию `sun`

# Координаты не сохраняются в UI
→ Возможно, они заданы в `configuration.yaml` - редактируйте только там
```

---

## Ссылки

- 🏠 [Home Assistant Location Docs](https://www.home-assistant.io/integrations/homeassistant/#general-settings)
- 📍 [Zone Configuration](https://www.home-assistant.io/integrations/zone/)
- 📱 [Companion App Location Tracking](https://companion.home-assistant.io/docs/core/location)
- 🗺️ [Get Coordinates (OpenStreetMap)](https://www.openstreetmap.org)
