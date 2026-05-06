---
author: ["Potato Energy Team", "ponfertato"]
categories: ["linux", "ventoy", "multiboot", "guide"]
date: "2026-05-06T12:35:00+03:00"
description: "Ventoy: одна флешка - десятки образов. Организация, темы, авто-установка Windows."
draft: false
series: ["Linux"]
slug: "ventoy-multiboot"
tags: ["ventoy", "multiboot", "windows", "linux", "automation", "grub"]
title: "Ventoy: Многозагрузочная флешка с правильной структурой"
---

Вместо того чтобы форматировать флешку под каждый новый образ, **Ventoy** позволяет просто копировать ISO-файлы как обычные файлы. При загрузке вы получаете меню со всеми доступными образами.

**Преимущества**:

- ✅ Не нужно перезаписывать флешку для каждого образа
- ✅ Поддержка Windows, Linux, утилит - всё на одном носителе
- ✅ Обычные файлы доступны из любой ОС
- ✅ Гибкая настройка через JSON-конфиги

---

## 🗂 Правильная структура флешки

```
[Первый раздел флешки - exFAT/NTFS]
├── /ventoy/                    # ← Обязательно здесь!
│   ├── ventoy.json            # Главный конфиг
│   ├── revi/                  # Авто-установка Windows
│   │   └── autounattend.xml
│   └── theme/                 # Кастомная тема
│       └── distro/theme.txt
│
├── BACKUP/                    # Рабочие файлы
├── LINUX/                     # Образы Linux
│   ├── Archlinux 2025.12.01.iso
│   ├── Debian 13.2.0.iso
│   └── NixOS 25.11.1734.iso
├── WINDOWS/                   # Образы Windows
│   ├── Windows 10 22H2.iso
│   ├── Windows 10 Enterprise LTSC 2021.iso
│   └── Windows 11 25H2.iso
├── ReviSetup/                 # Скрипты пост-установки
│   └── setup.cmd
└── UTILS/                     # Утилиты
    ├── gparted-live.iso
    └── memtest86+.iso
```

> ⚠️ **Критично**: `/ventoy/` должна быть на **первом разделе** (том, где лежат ISO), не в корне!

---

## ⚙️ Базовая конфигурация: `ventoy.json`

Файл должен лежать в `/ventoy/ventoy.json`, кодировка UTF-8.

### Минимальный пример

```json
{
  "control": [
    { "VTOY_DEFAULT_MENU_MODE": "1" },
    { "VTOY_FILT_DOT_UNDERSCORE_FILE": "1" }
  ],
  "theme": {
    "file": "/ventoy/theme/distro/theme.txt",
    "gfxmode": "1920x1080",
    "resolution_fit": "1"
  }
}
```

| Параметр                        | Описание                                 |
| ------------------------------- | ---------------------------------------- |
| `VTOY_DEFAULT_MENU_MODE`        | Режим меню по умолчанию                  |
| `VTOY_FILT_DOT_UNDERSCORE_FILE` | Скрывать файлы, начинающиеся с `.` и `_` |
| `theme.file`                    | Путь к файлу темы GRUB                   |
| `gfxmode`                       | Разрешение меню                          |
| `resolution_fit`                | Авто-подгонка под экран                  |

---

## 🎨 Кастомизация: темы для меню

### Где взять темы

1. **[distro-grub-themes](https://github.com/AdisonCavani/distro-grub-themes)** - коллекция готовых тем для Ventoy
2. **[Gnome-look.org](https://www.gnome-look.org/browse?cat=109)** - темы с тегом `ventoy`
3. **Своя тема** - создайте по документации Ventoy

### Установка темы

```bash
# Склонируйте тему
cd /mnt/ventoy/ventoy/theme
git clone https://github.com/AdisonCavani/distro-grub-themes.git distro

# Или скачайте с Gnome-look
wget https://www.gnome-look.org/.../theme.tar.gz
tar xzf theme.tar.gz
```

В `ventoy.json` укажите путь:

```json
{
  "theme": {
    "file": "/ventoy/theme/distro/theme.txt",
    "resolution_fit": "1"
  }
}
```

---

## 🪟 Авто-установка Windows

### Структура для автоустановки

```
/ventoy/
└── revi/
    └── autounattend.xml     # Ответы для Windows Setup

/ReviSetup/
└── setup.cmd                # Пост-установка
```

### Пример `ventoy.json` для автоустановки

```json
{
  "auto_install": [
    {
      "parent": "/WINDOWS",
      "template": ["/ventoy/revi/autounattend.xml"],
      "autosel": 1
    }
  ]
}
```

| Параметр   | Значение                  |
| ---------- | ------------------------- |
| `parent`   | Папка с образами Windows  |
| `template` | Путь к `autounattend.xml` |
| `autosel`  | Авто-выбор шаблона        |

### Что делает `autounattend.xml`

- Пропускает создание аккаунта Microsoft
- Отключает телеметрию
- Применяет настройки приватности
- Запускает `setup.cmd` после установки

> 💡 Готовые конфиги: [meetrevision/ventoy-conf](https://github.com/meetrevision/ventoy-conf)

---

## 🛠 Пост-установка: `setup.cmd`

Скрипт применяется автоматически после установки Windows.

### Пример действий

```cmd
:: Отключение обновлений драйверов
reg add "HKLM\Software\Policies\Microsoft\Windows\DriverSearching" /v "SearchOrderConfig" /t REG_DWORD /d 0 /f

:: Пауза обновлений до 2038
reg add "HKLM\SOFTWARE\Microsoft\WindowsUpdate\UX\Settings" /v "PauseUpdatesExpiryTime" /t REG_SZ /d "2038-01-19T03:14:07Z" /f

:: Отключение телеметрии
reg add "HKLM\Software\Policies\Microsoft\Windows\DataCollection" /v "AllowTelemetry" /t REG_DWORD /d 0 /f

:: Запрет автоустановки Teams
reg add "HKLM\SOFTWARE\Policies\Microsoft\Windows\Windows Chat" /v "ChatIcon" /t REG_DWORD /d 2 /f
```

---

## 🗂 Чеклист создания флешки

- [ ] Установил Ventoy через `Ventoy2Disk.sh` / `.exe`
- [ ] Создал `/ventoy/` на первом разделе
- [ ] Положил `ventoy.json` в `/ventoy/` (UTF-8)
- [ ] Скопировал образы в логические папки (`LINUX/`, `WINDOWS/`)
- [ ] (Опционально) Установил тему меню
- [ ] (Опционально) Настроил авто-установку Windows
- [ ] Проверил синтаксис JSON через [json.cn](https://www.json.cn)
- [ ] Протестировал загрузку

---

## 🔧 Полезные команды

```bash
# Проверка синтаксиса ventoy.json
cat /ventoy/ventoy.json | jq .

# В меню Ventoy: F5 - показать содержимое ventoy.json

# Пересоздать раздел Ventoy (данные будут удалены!)
sudo ./Ventoy2Disk.sh -i /dev/sdX
```

---

## 📚 Дополнительные возможности

Ventoy поддерживает множество плагинов и расширений:

- **Persistence** - сохранение данных между перезагрузками Live-систем
- **Memdisk** - загрузка образов в RAM
- **Auto install** - автоматизация установки ОС
- **Custom menu** - свои пункты меню
- **Injection** - внедрение драйверов/файлов в образы

> 🔗 **Документация**: [ventoy.net](https://www.ventoy.net/en/plugin_entry.html)

---

## ⚠️ Частые проблемы

| Симптом                      | Решение                                               |
| ---------------------------- | ----------------------------------------------------- |
| `ventoy.json` не применяется | Проверить путь: строго `/ventoy/ventoy.json`          |
| Тема не загружается          | Проверить кодировку (UTF-8) и путь в конфиге          |
| Автоустановка не работает    | Убедиться, что `parent` указывает на правильную папку |
| Ошибка парсинга JSON         | Проверить синтаксис через онлайн-валидатор            |

---

## 🔐 Безопасность

- ✅ exFAT - доступ из любой ОС (но без шифрования)
- 🔐 Для конфиденциальных файлов: VeraCrypt-контейнер
- ✅ `autounattend.xml` и `setup.cmd` - текстовые файлы, не содержат паролей

---

## 🗂 Ссылки

- 🚀 [Официальный сайт Ventoy](https://www.ventoy.net)
- 🎨 [Коллекция тем](https://github.com/AdisonCavani/distro-grub-themes)
- 🎨 [Темы на Gnome-look](https://www.gnome-look.org/browse?cat=109&tag=ventoy)
- 🪟 [Автоустановка Windows](https://github.com/meetrevision/ventoy-conf)
- 📘 [Документация плагинов](https://www.ventoy.net/en/plugin_entry.html)
