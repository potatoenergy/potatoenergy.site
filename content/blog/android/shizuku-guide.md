---
author: ["Potato Energy Team", "ponfertato"]
categories: ["android", "shizuku", "guide"]
date: "2026-03-20T13:15:00+03:00"
description: "Shizuku: системные возможности Android без root. Настройка и примеры с Obtainium и SAI."
draft: false
series: ["Android Tips"]
slug: "shizuku-guide"
tags: ["android", "shizuku", "obtainium", "sai", "adb", "system"]
title: "Shizuku: Системные возможности Android без root"
---

Shizuku - сервис, который даёт приложениям доступ к системным API Android **без root-прав**. Работает через ADB (Android Debug Bridge), используя привилегии `shell`.

**Зачем нужен**:

- Устанавливать приложения без подтверждения (Obtainium, SAI)
- Замораживать/размораживать приложения (Ice Box, Hail)
- Управлять разрешениями (AppOps, Permission Pilot)
- Менять настройки системы (DarQ, Naptime)
- Удалять системные приложения (Canta, AppManager)

> 💡 Shizuku не даёт полный root - только ограниченный доступ к системным функциям. Безопаснее, чем рут, но мощнее, чем обычное приложение.

---

## 📦 Установка и запуск

### Шаг 1: Скачать Shizuku

- [F-Droid](https://f-droid.org/packages/moe.shizuku.privileged.api/) (рекомендуется)
- [GitHub Releases](https://github.com/RikkaApps/Shizuku/releases)

### Шаг 2: Включить режим разработчика

1. Настройки → О телефоне → 7 раз тапнуть по «Версии MIUI» / «Номер сборки»
2. Вернуться в настройки → Дополнительные → **Для разработчиков**

### Шаг 3: Включить беспроводную отладку

1. В «Для разработчиков» → ✅ **Беспроводная отладка**
2. Нажать «Сопряжение по коду» → запомнить код и порт
3. В Shizuku: «Запустить» → «Сопряжение» → ввести код и порт
4. После сопряжения: «Запустить» → сервис запустится

**Проверить статус**:

```
Статус: Работает
Версия: 13.x.x
```

> ⚠️ После перезагрузки телефона Shizuku нужно запускать заново (процесс не сохраняется).

---

## 🔧 Пример 1: Obtainium + Shizuku

[Obtainium](https://github.com/ImranR98/Obtainium) - менеджер обновлений приложений напрямую из источников (GitHub, GitLab, F-Droid).

### Зачем Shizuku для Obtainium

| Без Shizuku                     | С Shizuku                    |
| ------------------------------- | ---------------------------- |
| Ручное подтверждение установки  | Автоматическая установка     |
| Не работает со split-APK        | Поддержка всех форматов      |
| Требует «Неизвестные источники» | Установка через системный PM |

### Настройка

1. Установить [Obtainium с F-Droid](https://f-droid.org/packages/dev.imranr.obtainium.fdroid/)
2. Открыть Obtainium → Настройки → **Метод установки**
3. Выбрать **Shizuku** (автоматически определится, если сервис запущен)
4. Добавить приложения для отслеживания:
   - Ввести URL репозитория: `https://github.com/user/repo`
   - Или выбрать из каталога
5. При обновлении: Obtainium скачает → установит через Shizuku → без подтверждения

**Пример добавления приложения**:

```
Источник: GitHub
URL: https://github.com/RikkaApps/Shizuku
Фильтр: Releases (stable)
Формат: APK
```

---

## 🔧 Пример 2: SAI + Shizuku

[SAI (Split APKs Installer)](https://github.com/Aefyr/SAI) - установщик split-APK, XAPK, APKS (форматы, которые не ставятся стандартным установщиком).

### Зачем Shizuku для SAI

| Без Shizuku                          | С Shizuku                             |
| ------------------------------------ | ------------------------------------- |
| Ручное подтверждение для каждого APK | Пакетная установка без подтверждения  |
| Не работает с некоторыми форматами   | Поддержка всех split-форматов         |
| Ошибки при установке                 | Надёжная установка через системный PM |

### Настройка

1. Установить [SAI с F-Droid](https://f-droid.org/packages/com.aefyr.sai.fdroid/)
2. Открыть SAI → Настройки → **Метод установки**
3. Выбрать **Shizuku** (или «Session API + Shizuku» для максимальной совместимости)
4. Установить приложение:
   - Нажать «Установить APK» → выбрать файл `.xapk`, `.apks`, `.apk`
   - SAI распакует → установит через Shizuku → готово

**Поддерживаемые форматы**:

- `.apk` - обычный пакет
- `.xapk` - APK + OBB-данные
- `.apks` / `.apk-m` - split-APK (несколько файлов для разных архитектур)

---

## 🔄 Автозапуск Shizuku (опционально)

После перезагрузки Shizuku останавливается. Варианты автозапуска:

### Вариант 1: Tasker + ADB (без root)

```bash
# Скрипт для Tasker: start-shizuku.sh
adb shell sh /sdcard/Android/data/moe.shizuku.privileged.api/start.sh
```

**Настройка**:

1. Установить [Tasker](https://play.google.com/store/apps/details?id=net.dinglisch.android.taskerm)
2. Создать задачу → «Run Shell» → команда выше
3. Триггер: «Device Boot»

### Вариант 2: KernelSU / Magisk (с root)

Если есть рут - установить Shizuku как системное приложение:

```bash
adb push shizuku.apk /data/local/tmp/
adb shell su -c "pm install /data/local/tmp/shizuku.apk"
adb shell su -c "sh /sdcard/Android/data/moe.shizuku.privileged.api/start.sh"
```

**Плюс**: Shizuku запускается автоматически при загрузке.

---

## 🔍 Диагностика

```bash
# Проверить, запущен ли Shizuku
adb shell sh /sdcard/Android/data/moe.shizuku.privileged.api/start.sh --check

# Посмотреть логи Shizuku
adb logcat | grep -i shizuku

# Проверить доступ приложения к Shizuku
adb shell dumpsys package moe.shizuku.privileged.api | grep -A5 "Granted permissions"
```

---

## ⚠️ Частые проблемы

```bash
# Shizuku не запускается
→ Перезапустить беспроводную отладку: выключить → включить
→ Перезагрузить телефон и запустить заново
→ Проверить, не блокирует ли антивирус/оптимизатор

# Obtainium/SAI не видят Shizuku
→ Убедиться, что сервис запущен (статус «Работает» в приложении)
→ Перезапустить Shizuku и целевое приложение
→ Проверить разрешения: Настройки → Приложения → [App] → Разрешения

# Ошибка установки «Package parser error»
→ Файл повреждён - перекачать
→ Неподдерживаемый формат - проверить версию SAI
→ Недостаточно места - очистить кэш

# Shizuku отключается сам
→ Настройки → Батарея → [Shizuku] → ✅ Без ограничений
→ Настройки → Приложения → [Shizuku] → ✅ Автозапуск
```

---

## 🛡 Безопасность

### Что может делать приложение с доступом к Shizuku

| Действие                      | Риск                                                         |
| ----------------------------- | ------------------------------------------------------------ |
| Установить/удалить приложение | Средний (требует подтверждения пользователя в Obtainium/SAI) |
| Изменить разрешения           | Средний (только для своего пакета или с явного согласия)     |
| Прочитать логи                | Низкий (только свои логи)                                    |
| Получить доступ к файлам      | Низкий (только с явного разрешения)                          |

### Как минимизировать риски

1. Устанавливать Shizuku только с [F-Droid](https://f-droid.org/) или [GitHub](https://github.com/RikkaApps/Shizuku)
2. Предоставлять доступ к Shizuku только доверенным приложениям (Obtainium, SAI, AppManager)
3. Останавливать Shizuku, когда не используется
4. Не включать беспроводную отладку в публичных сетях

---

## Ссылки

- 🔧 [Shizuku на F-Droid](https://f-droid.org/packages/moe.shizuku.privileged.api/)
- 📦 [Obtainium на F-Droid](https://f-droid.org/packages/dev.imranr.obtainium.fdroid/)
- 📱 [SAI на F-Droid](https://f-droid.org/packages/com.aefyr.sai.fdroid/)
- 📘 [Shizuku Documentation](https://shizuku.rikka.app/)
- 🔐 [Android ADB Docs](https://developer.android.com/tools/adb)
