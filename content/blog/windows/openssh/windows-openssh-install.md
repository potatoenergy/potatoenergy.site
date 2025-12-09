---
author: ["Potato Energy Team", "ponfertato"]
categories: ["SSH"]
date: "2024-03-31T00:00:00+03:00"
description: ""
draft: true
series: ["SSH", "OpenSSH"]
slug: "install"
tags: ["windows", "ssh", "openssh"]
title: "Установка и настройка OpenSSH сервера в Windows"
---

**OpenSSH** - это мощное средство для обеспечения безопасного удаленного подключения, которое использует протокол SSH (Secure Shell). С помощью OpenSSH ты можешь устанавливать защищенные соединения между клиентом и сервером, обеспечивая конфиденциальность и целостность передаваемых данных.

> ***Почему это так важно?*** \
OpenSSH шифрует весь трафик между клиентом и сервером, что делает его непригодным для перехвата информации или атак типа "man-in-the-middle". Это означает, что даже если злоумышленники попытаются перехватить данные или попытаться подделать подключение, OpenSSH обеспечивает надежную защиту.

Кроме того, OpenSSH предлагает различные функции аутентификации, такие как использование ключей SSH, что делает процесс аутентификации более безопасным и удобным. Это позволяет настроить доступ к серверу только для авторизованных пользователей, чтобы гарантировать безопасность системы.

# Требования

1. Операционная система: Windows Server 2019 или Windows 10 (сборка 1809). OpenSSH сервер может быть установлен на версии Pro, Enterprise и Education. Важно, чтобы на компьютере была установлена одна из этих версий.

2. Доступ к интернету: Компьютер должен иметь доступ к Интернету или локальной сети для корректной работы OpenSSH сервера.

3. Административные привилегии: Для установки и настройки OpenSSH сервера потребуются административные привилегии на компьютере. Убедитесь, что у вас есть соответствующие права доступа.

4. Установка OpenSSH: Для установки OpenSSH сервера на Windows следует скачать установщик OpenSSH с официального сайта Microsoft или из источников, доступных в Интернете.

5. Ресурсы компьютера: OpenSSH сервер не требует значительных ресурсов компьютера. Однако, убедитесь, что на вашем компьютере достаточно процессорной мощности и оперативной памяти для обеспечения плавной работы сервера.

# Включение необходимых компонентов

## Через командную строку

Пакет сервера OpenSSH является частью всех современных версий Windows 10 (начиная с 1803), Windows 11 и Windows Server 2022/2019 как Feature on Demand (FoD). Чтобы установить сервер OpenSSH, выполните следующие шаги:

- Сначала откройте повышенную командную строку PowerShell. Для этого щелкните правой кнопкой мыши по значку "Пуск" и выберите "Windows PowerShell (Администратор)".

- В командной строке PowerShell введите следующую команду:

```ps1
Get-WindowsCapability -Online | Where-Object Name -like 'OpenSSH.Server*' | Add-WindowsCapability -Online
```

![](img/content/blog/windows/openssh/WindowsTerminal_X4D0Pdn6r8.png)
Эта команда просмотрит доступные возможности Windows и установит пакет сервера OpenSSH.
![](img/content/blog/windows/openssh/WindowsTerminal_Y8mLOC5fu7.png)

- Если вы предпочитаете использовать команду DISM, введите следующую команду:

```ps1
dism /Online /Add-Capability /CapabilityName:OpenSSH.Server~~~~0.0.1.0
```

Эта команда также активирует сервер OpenSSH на вашей системе.
![](img/content/blog/windows/openssh/WindowsTerminal_b8nMmitCri.png)

## Через параметры системы

Ты также можешь установить OpenSSH на Windows 10/11 с помощью современной панели настроек (Настройки -> Приложения и компоненты -> Дополнительные компоненты -> Добавить компонент). Найди в списке функцию "Сервер OpenSSH" и нажми "Установить".
![](img/content/blog/windows/openssh/mstsc_BRQMECACUC.png)

## Через MSI установщик

Один из вариантов - это использование MSI-установщика, который можно найти в официальном репозитории Microsoft на GitHub (<https://github.com/PowerShell/Win32-OpenSSH/releases/>). Там необходимо скачать MSI-пакет OpenSSH-Win64-v*.*.*.*.msi для Windows (Актуальный на момент написания статьи релиз: v9.5.0.0p1-Beta). Просто загрузить его и установить на свой компьютер следующими шагами:

1. Загрузите MSI-файл, используя следующую команду PowerShell:

```ps1
Invoke-WebRequest https://github.com/PowerShell/Win32-OpenSSH/releases/download/v9.5.0.0p1-Beta/OpenSSH-Win64-v9.5.0.0.msi -OutFile $HOME\Downloads\OpenSSH-Win64-v9.5.0.0.msi -UseBasicParsing
```

2. Затем выполним команду установки MSI для установки OpenSSH клиента и сервера:

```batch
msiexec /i $HOME\Downloads\OpenSSH-Win64-v9.5.0.0.msi
```

После завершения установки сервера OpenSSH вы сможете настроить его и использовать для выполнения различных задач связанных с удаленным доступом, безопасным обменом данными и управлением учетными записями.

# Настройка необходимых компонентов

После установки сервера OpenSSH на Windows, добавляются две службы:

- ssh-agent (OpenSSH Authentication Agent) - можно использовать для управления приватными ключами, если настроена аутентификация по SSH-ключам.
- sshd (OpenSSH Server).

Для настройки сервера OpenSSH на Windows, выполните следующие шаги:

Измените тип запуска службы sshd на автоматический, используя PowerShell:

```ps1
Set-Service -Name sshd -StartupType 'Automatic'
```

Запустите службу sshd с помощью команды PowerShell:

```ps1
Start-Service sshd
```

Проверьте, работает ли SSH-сервер на Windows и ожидает подключений на TCP-порту 22, используя команду netstat:

```batch
netstat -na | find ":22"
```

Убедитесь, что брандмауэр Windows Defender разрешает входящие соединения на TCP-порту 22, используя команду PowerShell:

```ps1
Get-NetFirewallRule -Name *OpenSSH-Server* | select Name, DisplayName, Description, Enabled
```

Если правило отключено (Enabled=False) или отсутствует, вы можете создать новое правило входящего соединения с использованием командлета New-NetFirewallRule, например:

```ps1
New-NetFirewallRule -Name sshd -DisplayName 'OpenSSH Server (sshd)' -Enabled True -Direction Inbound -Protocol TCP -Action Allow -LocalPort 22
```

По умолчанию, ключевые компоненты OpenSSH располагаются в следующих папках:

Исполняемые файлы сервера OpenSSH: C:\Windows\System32\OpenSSH (sshd.exe, ssh.exe, ssh-keygen.exe, sftp.exe и пр.)

- Файл sshd_config (создается после первого запуска службы sshd): `C:\ProgramData\ssh`
- Файл authorized_keys и ключи могут храниться в папке профиля пользователя: `%USERPROFILE%\.ssh\`
