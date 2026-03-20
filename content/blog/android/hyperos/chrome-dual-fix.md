---
author: ["Potato Energy Team", "ponfertato"]
categories: ["android", "hyperos", "guide"]
date: "2026-03-20T13:25:00+03:00"
description: "Удаление Google Chrome из второго пространства на HyperOS (Xiaomi/POCO). Решения через Shizuku, AppManager и ADB."
draft: false
series: ["Android Tips"]
slug: "chrome-dual-fix"
tags: ["android", "hyperos", "chrome", "shizuku", "appmanager", "dual-apps"]
title: "HyperOS: Удаление Chrome из второго пространства"
---

## Проблема

После перехода на HyperOS (POCO, Xiaomi) ссылки из приложений (Telegram, WhatsApp и др.) открываются не в основном аккаунте Chrome, а в клонированном - даже если в настройках системы выбран основной браузер.

**Симптомы**:

- Клик по ссылке → открывается Chrome второго пространства
- В настройках «Приложения по умолчанию» выбран основной Chrome
- Сброс настроек не помогает

**Причина**: HyperOS приоритизирует клонированные приложения при обработке интентов, игнорируя выбор пользователя.

> 💡 Проблема воспроизводится на MIUI 14 / HyperOS 1.0+ с включённой функцией «Клонирование приложений» / «Второе пространство».

---

## ❌ Почему простые решения не работают

### Удаление через ADB (временное)

```bash
adb shell
pm uninstall -k --user 999 com.android.chrome
```

**Проблема**: после перезагрузки системы Chrome во втором пространстве **тихо переустанавливается**. `user 999` - это ID клонированного профиля, но HyperOS восстанавливает системные приложения при старте.

### Отключение «Клонирования приложений»

**Минус**: удаляет **все** клонированные приложения и их данные. Не подходит, если нужны другие дубликаты (мессенджеры, банки).

---

## ✅ Решения (по предпочтению)

### Решение 1: Shizuku + AppManager (рекомендуется)

Позволяет удалить Chrome из второго пространства **без root**, с сохранением остальных клонированных приложений.

#### Шаг 1: Установить Shizuku

1. Скачать [Shizuku с F-Droid](https://f-droid.org/packages/moe.shizuku.privileged.api/)
2. Запустить приложение → «Запустить через беспроводную отладку»
3. Включить беспроводную отладку в «Для разработчиков»:
   - Настройки → О телефоне → 7 раз тапнуть по «Версии ОС»
   - Настройки → Дополнительные → Для разработчиков → ✅ Беспроводная отладка
4. Следовать инструкциям в Shizuku для сопряжения

#### Шаг 2: Установить AppManager

1. Скачать [AppManager с F-Droid](https://f-droid.org/packages/io.github.muntashirakon.AppManager/)
2. Открыть приложение → предоставить доступ через Shizuku (автоматически)

#### Шаг 3: Удалить Chrome из второго пространства

1. Найти `com.android.chrome` (Google Chrome)
2. Открыть карточку приложения → ⋮ → **Удалить для пользователя**
3. Подтвердить удаление

**Результат**: Chrome удалён только из второго пространства, основной аккаунт не затронут.

#### Шаг 4: Проверить

```bash
# Через ADB (опционально)
adb shell pm list packages --user 999 | grep chrome
# Должно быть пусто
```

---

### Решение 2: Сменить браузер по умолчанию

Если не хочется настраивать Shizuku:

1. Установить альтернативный браузер: [Chrome Beta](https://play.google.com/store/apps/details?id=com.chrome.beta), Firefox, Brave
2. Экспортировать закладки из основного Chrome (синхронизация с аккаунтом)
3. Очистить данные основного Chrome (опционально)
4. Назначить новый браузер по умолчанию:
   - Настройки → Приложения → Приложения по умолчанию → Браузер
5. Проверить открытие ссылок

**Плюс**: не требует root, Shizuku или ADB  
**Минус**: нужно привыкнуть к новому браузеру

---

### Решение 3: ADB-скрипт с автозапуском (для продвинутых)

Если Shizuku не работает, можно автоматизировать ADB-команды:

```bash
#!/bin/bash
# remove-chrome-clone.sh
adb wait-for-device
adb shell pm uninstall -k --user 999 com.android.chrome
echo "Chrome удалён из второго пространства"
```

**Автозапуск через Termux + ADB**:

1. Установить [Termux](https://f-droid.org/packages/com.termux/)
2. Установить [ADB Keyboard](https://f-droid.org/packages/de.stefanbechtold.simpleterm/) или использовать `adb tcpip`
3. Запустить скрипт при загрузке через `~/.termux/boot/`

**Ограничение**: после перезагрузки телефона нужно запускать скрипт вручную (или настраивать автозапуск через Tasker).

---

## 🔍 Диагностика

```bash
# Проверить, установлен ли Chrome во втором пространстве
adb shell pm list packages --user 999 | grep chrome

# Проверить браузер по умолчанию
adb shell dumpsys package preferred | grep browser

# Посмотреть обработчики интентов для ссылок
adb shell dumpsys activity preferred-activities | grep -A5 "http"
```

---

## ⚠️ Частые проблемы

```bash
# Shizuku не запускается
→ Включить беспроводную отладку и сопряжение заново
→ Перезапустить Shizuku после обновления системы

# AppManager не видит пользователя 999
→ Предоставить права через Shizuku: Настройки → Приложения → AppManager → Разрешения
→ Перезапустить AppManager

# Chrome переустанавливается после ребута
→ Это особенность HyperOS. Решение: добавить скрипт в автозагрузку (Termux + Tasker)
→ Или использовать Shizuku + AppManager с повторным удалением при загрузке (требует рут)

# Ссылки всё равно открываются в клоне
→ Очистить настройки по умолчанию: Настройки → Приложения → Управление приложениями → ⋮ → Сбросить настройки
→ Перезапустить телефон
```

---

## Ссылки

- 🔧 [Shizuku на F-Droid](https://f-droid.org/packages/moe.shizuku.privileged.api/)
- 📱 [AppManager на F-Droid](https://f-droid.org/packages/io.github.muntashirakon.AppManager/)
- 🤖 [Termux на F-Droid](https://f-droid.org/packages/com.termux/)
- 📘 [Android Package Manager Docs](https://developer.android.com/tools/adb#pm)
