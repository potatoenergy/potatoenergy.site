---
author: ["Potato Energy Team", "ponfertato"]
categories: ["windows", "linux", "wsl", "tutorial"]
date: "2026-03-16T22:00:00+03:00"
description: "WSL2 на практике: установка, настройка, интеграция с Windows. Гайд для разработчиков."
draft: false
series: ["Windows Tips"]
slug: "wsl"
tags: ["windows", "wsl", "linux", "docker", "dev"]
title: "WSL2: Полный гайд для разработчика"
---

WSL (Windows Subsystem for Linux) - подсистема для запуска нативных Linux-приложений прямо в Windows. Без виртуальной машины, без двойной загрузки.

**WSL1** - трансляция системных вызовов (быстро, но не 100% совместимо)  
**WSL2** - полноценное ядро Linux в лёгкой виртуализации (полная совместимость, чуть больше ресурсов)

> 💡 Используй WSL2. Почти без накладных расходов, но с полной поддержкой Docker, systemd и всех фич Linux.

---

## Требования

- **ОС**: Windows 10 (2004+, сборка 19041+) или Windows 11
- **Архитектура**: x64 или ARM64
- **Права**: Администратор (для установки)
- **Виртуализация**: Включена в BIOS/UEFI (Hyper-V Platform)

**Проверить виртуализацию:**

```powershell
systeminfo | findstr /I "Virtualization"
# Должно быть: "Гипервизор обнаружен" или "Virtualization Enabled"
```

---

## Установка (один командой)

```powershell
# Запусти PowerShell от имени Администратора
wsl --install
```

Это сделает всё:

- ✅ Включит компоненты WSL и VirtualMachinePlatform
- ✅ Скачает и установит Ubuntu (дистрибутив по умолчанию)
- ✅ Настроит WSL2 как версию по умолчанию

**Перезагрузи компьютер** после выполнения.

---

## Установка вручную (если `--install` не работает)

```powershell
# 1. Включить компоненты
dism /online /enable-feature /featurename:Microsoft-Windows-Subsystem-Linux /all /norestart
dism /online /enable-feature /featurename:VirtualMachinePlatform /all /norestart

# 2. Перезагрузиться
shutdown /r /t 0

# 3. Скачать и установить ядро WSL2
# https://aka.ms/wsl2kernel

# 4. Задать WSL2 по умолчанию
wsl --set-default-version 2

# 5. Установить дистрибутив из Microsoft Store
# Или через winget:
winget install Ubuntu.Ubuntu.24.04
```

---

## Первый запуск

```bash
# После перезагрузки откроется терминал с установкой Linux
# Задай имя пользователя и пароль (не отображается при вводе - это нормально)

# Обновить пакеты (Ubuntu/Debian)
sudo apt update && sudo apt upgrade -y

# Установить базовые инструменты
sudo apt install -y git curl wget build-essential
```

---

## Управление дистрибутивами

```powershell
# Список установленных
wsl --list --verbose
# или кратко:
wsl -l -v

# Запустить конкретный дистрибутив
wsl -d Ubuntu-24.04
wsl -d Debian

# Остановить (выключить) дистрибутив
wsl --terminate Ubuntu-24.04

# Остановить все
wsl --shutdown

# Задать версию (1 или 2)
wsl --set-version Ubuntu-24.04 2

# Задать дистрибутив по умолчанию
wsl --set-default Ubuntu-24.04

# Экспорт / импорт
wsl --export Ubuntu-24.04 D:\backups\ubuntu.tar
wsl --import MyUbuntu D:\wsl\MyUbuntu D:\backups\ubuntu.tar --version 2

# Удалить дистрибутив (все данные будут потеряны!)
wsl --unregister Ubuntu-24.04
```

---

## Доступ к файлам

### Из Linux → в Windows

```bash
# Windows-диски смонтированы в /mnt/
ls /mnt/c/Users/ponfertato/Documents

# Открыть проводник в текущей папке
explorer.exe .
```

### Из Windows → в Linux

```powershell
# Открыть домашнюю папку пользователя в проводнике
\\wsl$\Ubuntu-24.04\home\kirill

# Или из терминала:
explorer.exe \\wsl$\Ubuntu-24.04\home\kirill
```

> ⚠️ Не редактируй Linux-файлы из Windows-приложений напрямую (через `\\wsl$`). Это может повредить метаданные. Используй Linux-инструменты внутри WSL.

---

## Сеть и порты

```bash
# Узнать свой IP в WSL
ip addr show eth0 | grep inet

# Windows и WSL используют общую сеть (localhost пробрасывается)
# Запусти веб-сервер в WSL:
python3 -m http.server 8000

# Доступен из Windows по:
# http://localhost:8000
```

**Если порты не пробрасываются:**

```powershell
# Проверить, включена ли настройка
wsl --status

# Включить автоматический проброс (если отключён)
# В %USERPROFILE%\.wslconfig:
[wsl2]
networkingMode=mirrored
localhostForwarding=true
```

---

## Интеграция с терминалом и редактором

### Использовать Windows Terminal

```powershell
# Установить из Microsoft Store или:
winget install Microsoft.WindowsTerminal

# WSL-профили добавляются автоматически
# Переключайся между оболочками: Ctrl+Shift+1/2/3...
```

### VS Code + WSL

```bash
# Внутри WSL:
code .

# Откроет VS Code на Windows с подключением к WSL-окружению
# Установи расширение "Remote - WSL" если потребуется
```

### Доступ к Windows-программам из WSL

```bash
# Запустить Notepad из Linux:
notepad.exe /mnt/c/Users/ponfertato/notes.txt

# Запустить PowerShell:
powershell.exe Get-Process
```

---

## Docker в WSL2

```powershell
# Установить Docker Desktop для Windows
winget install Docker.DockerDesktop

# В настройках Docker Desktop:
# Settings → Resources → WSL Integration → включить твой дистрибутив

# Проверить в WSL:
docker run hello-world
```

> 💡 Docker работает нативно в WSL2 - без дополнительной настройки внутри Linux.

---

## systemd в WSL (для сервисов)

```bash
# Включить systemd (WSL 0.67.6+)
# Создать/отредактировать: /etc/wsl.conf
[boot]
systemd=true

# Применить:
# Из PowerShell:
wsl --terminate Ubuntu-24.04
wsl -d Ubuntu-24.04

# Проверить:
systemctl status
```

Теперь работают: `systemctl start nginx`, `enable docker` и другие сервисы.

---

## Полезные настройки (.wslconfig)

Файл: `%USERPROFILE%\.wslconfig` (создай, если нет)

```ini
[wsl2]
# Память: 4 ГБ (или 50% от ОЗУ)
memory=4GB
# Процессоры: 4 ядра
processors=4
# Своп: 2 ГБ
swap=2GB
# Диск: динамический, до 256 ГБ
diskSize=256GB
# Сеть: режим зеркалирования (лучшая совместимость)
networkingMode=mirrored
# Автозапуск: выключить
autoShutdown=true
# Таймаут бездействия: 10 минут
idleTimeout=600000
```

Применить: `wsl --shutdown`, затем запустить дистрибутив снова.

---

## Частые проблемы

```powershell
# WSL не запускается, ошибка 0x80370102
→ Включи виртуализацию в BIOS
→ Проверь: "Панель управления" → "Программы" → "Включение компонентов Windows" → Hyper-V Platform

# Ошибка "Виртуальная машина не запустилась"
→ Запусти от администратора:
  bcdedit /set hypervisorlaunchtype auto
→ Перезагрузись

# Медленный доступ к файлам в /mnt/c
→ Храни проекты внутри Linux-файловой системы: ~/projects, а не в /mnt/c/...
→ Используй VS Code Remote WSL для редактирования

# Не работает sudo / пароль не принимается
→ Сброс пароля:
  # В PowerShell:
  wsl -d Ubuntu-24.04 -u root
  # Внутри WSL:
  passwd kirill

# Конфликт портов с Windows
→ Проверь, что не занято:
  netsh interface ipv4 show excludedportrange protocol=tcp
→ Или смени порт в приложении
```

---

## Бэкап и миграция

```powershell
# Экспорт дистрибутива
wsl --export Ubuntu-24.04 D:\wsl-backups\ubuntu-$(Get-Date -Format 'yyyy-MM-dd').tar

# Импорт на другом ПК
wsl --import Ubuntu-24.04 D:\WSL\Ubuntu D:\wsl-backups\ubuntu-2026-03-16.tar --version 2

# Настроить пользователя по умолчанию после импорта
# Создать: /etc/wsl.conf в импортированном дистрибутиве
[user]
default=kirill
```

---

## Ссылки

- 🌐 [Официальная документация WSL](https://learn.microsoft.com/ru-ru/windows/wsl/)
- 📦 [Дистрибутивы в Microsoft Store](https://aka.ms/wslstore)
- ⚙️ [Настройка .wslconfig](https://learn.microsoft.com/ru-ru/windows/wsl/wsl-config)
- 🐳 [Docker + WSL2](https://docs.docker.com/desktop/wsl/)
