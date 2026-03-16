---
author: ["Potato Energy Team", "ponfertato"]
categories: ["ssh", "windows", "tutorial"]
date: "2026-03-16T15:25:00+03:00"
description: "Установка и настройка OpenSSH сервера на Windows 10/11 и Server 2019+. Быстро, через PowerShell."
draft: false
series: ["SSH"]
slug: "openssh"
tags: ["windows", "ssh", "openssh", "powershell", "server"]
title: "OpenSSH на Windows: Установка сервера"
---

OpenSSH - инструмент для безопасного удалённого доступа по протоколу SSH. Шифрует весь трафик, поддерживает аутентификацию по ключам, работает на Windows, Linux, macOS.

> 💡 После настройки ты сможешь подключаться к своему Windows-ПК как к обычному Linux-серверу: `ssh user@192.168.1.100`

---

## Требования

- **ОС**: Windows 10 (1809+), Windows 11, Windows Server 2019/2022
- **Права**: Администратор
- **Сеть**: Доступ к порту 22 (локально или извне)

---

## Установка (3 способа)

### Способ 1: PowerShell (рекомендуется)

```powershell
# Запусти от имени Администратора
# Установить OpenSSH сервер
Add-WindowsCapability -Online -Name OpenSSH.Server~~~~0.0.1.0

# Проверить установку
Get-WindowsCapability -Online | Where-Object Name -like 'OpenSSH*'
```

### Способ 2: DISM (альтернатива)

```powershell
dism /Online /Add-Capability /CapabilityName:OpenSSH.Server~~~~0.0.1.0
```

### Способ 3: Через настройки (GUI)

1. Настройки → Приложения → Дополнительные компоненты
2. «Добавить компонент» → найди «OpenSSH Server» → Установить

---

## Настройка службы

```powershell
# Запустить от имени Администратора

# Включить автозапуск sshd
Set-Service -Name sshd -StartupType Automatic

# Запустить службу
Start-Service sshd

# Проверить статус
Get-Service sshd

# Убедиться, что порт 22 слушается
netstat -ano | findstr :22
```

---

## Брандмауэр

```powershell
# Проверить правило для OpenSSH
Get-NetFirewallRule -Name *OpenSSH-Server* | Select Name, Enabled

# Если правила нет - создать
New-NetFirewallRule -Name sshd `
  -DisplayName 'OpenSSH Server' `
  -Enabled True `
  -Direction Inbound `
  -Protocol TCP `
  -LocalPort 22 `
  -Action Allow `
  -Profile Any
```

---

## Проверка подключения

```powershell
# С этого же ПК
ssh localhost

# С другого устройства в сети
ssh <твой_пользователь>@<IP_адрес_Windows>

# Пример:
ssh kirill@192.168.1.100
```

> 💡 Первый вход запросит подтверждение отпечатка ключа - введи `yes`.

---

## Аутентификация по ключам (рекомендуется)

### На клиенте (где подключаешься)

```bash
# Создать пару ключей (если нет)
ssh-keygen -t ed25519 -C "kirill@potatoenergy.ru"

# Скопировать публичный ключ на сервер
# Для Windows-сервера - вручную:
type $env:USERPROFILE\.ssh\id_ed25519.pub
# Скопируй вывод
```

### На сервере (Windows)

```powershell
# Создать папку .ssh в профиле пользователя
mkdir $env:USERPROFILE\.ssh -Force

# Создать/отредактировать authorized_keys
notepad $env:USERPROFILE\.ssh\authorized_keys

# Вставить публичный ключ (одной строкой), сохранить

# Задать правильные права (ОБЯЗАТЕЛЬНО)
$acl = Get-Acl $env:USERPROFILE\.ssh\authorized_keys
$acl.SetAccessRuleProtection($true, $false)
$rule = New-Object System.Security.AccessControl.FileSystemAccessRule(
    $env:USERNAME, "Read", "Allow")
$acl.AddAccessRule($rule)
Set-Acl $env:USERPROFILE\.ssh\authorized_keys $acl
```

### Отключить вход по паролю (опционально, для безопасности)

```powershell
# Отредактировать конфиг
notepad C:\ProgramData\ssh\sshd_config

# Найти и изменить строки:
# PasswordAuthentication no
# PubkeyAuthentication yes

# Перезапустить службу
Restart-Service sshd
```

---

## Конфигурация: полезные настройки

Файл: `C:\ProgramData\ssh\sshd_config`

```ini
# Только определённые пользователи
AllowUsers kirill admin

# Сменить порт (если 22 занят)
Port 2222

# Запретить root-логин
PermitRootLogin no

# Таймаут неактивности
ClientAliveInterval 300
ClientAliveCountMax 2

# Логирование
LogLevel VERBOSE
```

После правок:

```powershell
Restart-Service sshd
```

---

## Частые проблемы

```powershell
# Служба не запускается
→ Проверь логи: Get-WinEvent -LogName "OpenSSH/Operational" -MaxEvents 10

# Порт 22 не слушается
→ Проверь брандмауэр: Get-NetFirewallRule -Name sshd
→ Проверь, что служба запущена: Get-Service sshd

# "Permission denied (publickey,password)"
→ Проверь права на authorized_keys (должен читать только владелец)
→ Убедись, что публичный ключ вставлен одной строкой, без переносов

# Подключение есть, но нет прав на файлы
→ Проверь права пользователя на папки в Windows
→ Попробуй запустить терминал от имени администратора на клиенте
```

---

## Ссылки

- 🌐 [Официальный репозиторий Win32-OpenSSH](https://github.com/PowerShell/Win32-OpenSSH)
- 📘 [Документация Microsoft](https://learn.microsoft.com/ru-ru/windows-server/administration/openssh/openssh_install_firstuse)
- 🔑 [Генератор конфигов sshd](https://www.ssh.com/academy/ssh/sshd_config)
