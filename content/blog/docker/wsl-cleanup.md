---
author: ["Potato Energy Team", "ponfertato"]
categories: ["docker", "windows", "wsl", "guide"]
date: "2026-03-17T09:30:00+03:00"
description: "Очистка Docker Desktop на WSL2: освобождение места, оптимизация VHDX, автоматизация."
draft: false
series: ["Docker"]
slug: "docker-wsl-cleanup"
tags: ["docker", "wsl", "windows", "powershell", "cleanup"]
title: "Docker WSL: Очистка и оптимизация дисков"
---

Docker Desktop на Windows использует WSL2 с динамическими VHDX-файлами. Они **растут**, когда добавляются образы/контейнеры, но **не сжимаются автоматически** при удалении. Результат: диск `C:` заполняется, хотя `docker system df` показывает свободное место.

> 💡 Решение: ручная оптимизация VHDX через PowerShell + утилита Docker.

---

## 📦 Скрипт очистки

### Создание файла

```powershell
# Файл: $HOME\Scripts\docker-clear-wsl.ps1
$script = @'
$LOCAL = "$env:LOCALAPPDATA\Docker\wsl"
$VHD1 = Join-Path $LOCAL "disk\docker_data.vhdx"
$VHD2 = Join-Path $LOCAL "main\ext4.vhdx"

# 1. Очистка Docker
docker system prune -f

# 2. Возврат места через официальный инструмент
docker run --rm --privileged --pid=host docker/desktop-reclaim-space
docker rmi docker/desktop-reclaim-space -f

# 3. Остановка Docker Desktop
Get-Process -Name "Docker Desktop","com.docker.backend","com.docker.build" `
  -ErrorAction SilentlyContinue | Stop-Process -Force -ErrorAction SilentlyContinue

# 4. Выключение WSL
wsl --shutdown

# 5. Оптимизация VHDX-файлов (сжатие)
if (Test-Path $VHD1) { Optimize-VHD -Path $VHD1 -Mode Full }
if (Test-Path $VHD2) { Optimize-VHD -Path $VHD2 -Mode Full }

# 6. Запуск Docker Desktop
Start-Sleep -Seconds 2
Start-Process -FilePath "$env:ProgramFiles\Docker\Docker\Docker Desktop.exe" `
  -ErrorAction SilentlyContinue
'@

$script | Out-File -FilePath "$HOME\Scripts\docker-clear-wsl.ps1" -Encoding UTF8
```

### Запуск

```powershell
# От имени Администратора (требуется для Optimize-VHD)
powershell.exe -ExecutionPolicy Bypass -File "$HOME\Scripts\docker-clear-wsl.ps1"
```

---

## 🔍 Как это работает

| Шаг | Команда                        | Что делает                                                       |
| --- | ------------------------------ | ---------------------------------------------------------------- |
| 1   | `docker system prune -f`       | Удаляет остановленные контейнеры, неиспользуемые образы, кэш     |
| 2   | `docker/desktop-reclaim-space` | Официальный инструмент Docker для возврата места в WSL2          |
| 3   | `Stop-Process`                 | Корректно останавливает Docker Desktop (иначе VHDX заблокирован) |
| 4   | `wsl --shutdown`               | Полностью выключает WSL, освобождая файлы для оптимизации        |
| 5   | `Optimize-VHD -Mode Full`      | Сжимает VHDX-файлы, возвращая место на хосте                     |
| 6   | `Start-Process`                | Запускает Docker Desktop обратно                                 |

**Почему именно так**:

- Без остановки процессов `Optimize-VHD` не сработает (файл занят)
- `docker/desktop-reclaim-space` работает внутри WSL, удаляя «призрачные» данные
- `Mode Full` — максимальное сжатие (медленнее, но эффективнее)

---

## ⚙️ Автоматизация

### Через Планировщик заданий (GUI)

1. Открой **Task Scheduler** → Create Basic Task
2. Trigger: Weekly (например, воскресенье, 03:00)
3. Action: Start a program
   - Program: `powershell.exe`
   - Arguments: `-ExecutionPolicy Bypass -File "C:\Users\ponfertato\Scripts\docker-clear-wsl.ps1"`
4. Settings: ✅ Run with highest privileges

### Через PowerShell (скриптом)

```powershell
# Создать задачу от имени Администратора
$action = New-ScheduledTaskAction -Execute "powershell.exe" `
  -Argument "-ExecutionPolicy Bypass -File `"$HOME\Scripts\docker-clear-wsl.ps1`""

$trigger = New-ScheduledTaskTrigger -Weekly -DaysOfWeek Sunday -At 3am

$principal = New-ScheduledTaskPrincipal -UserId "SYSTEM" -RunLevel Highest

Register-ScheduledTask -TaskName "Docker WSL Cleanup" `
  -Action $action -Trigger $trigger -Principal $principal
```

### Проверка и запуск вручную

```powershell
# Запустить задачу
Start-ScheduledTask -TaskName "Docker WSL Cleanup"

# Проверить статус
Get-ScheduledTask -TaskName "Docker WSL Cleanup" | Get-ScheduledTaskInfo

# Просмотреть историю
Get-WinEvent -FilterHashtable @{LogName='Microsoft-Windows-TaskScheduler/Operational'; TaskName='Docker WSL Cleanup'} -MaxEvents 10
```

---

## 📊 Мониторинг результата

### До и после

```powershell
# Размер VHDX до очистки
Get-ChildItem "$env:LOCALAPPDATA\Docker\wsl\disk\docker_data.vhdx" | Select Name, @{N="SizeGB";E={[math]::Round($_.Length/1GB,2)}}

# После очистки — сравнить значение
```

### Статистика Docker

```powershell
# Общая статистика
docker system df

# Детализация
docker system df -v
```

**Ожидаемый результат**:

- `docker system df` покажет меньше данных
- Размер `.vhdx` на диске уменьшится (иногда в 2-5 раз)

---

## ⚠️ Безопасность и нюансы

### Требования

| Требование                            | Почему важно                               |
| ------------------------------------- | ------------------------------------------ |
| Запуск от Администратора              | `Optimize-VHD` требует повышенных прав     |
| Закрытые приложения, использующие WSL | Иначе `wsl --shutdown` не выполнится       |
| Резервная копия критичных данных      | На случай сбоя (маловероятно, но возможно) |

### Если что-то пошло не так

```powershell
# Docker не запускается после очистки
→ Запусти вручную: "$env:ProgramFiles\Docker\Docker\Docker Desktop.exe"
→ Проверь логи: Get-Content "$env:LOCALAPPDATA\Docker\log.txt" -Tail 50

# VHDX не сжался
→ Убедись, что Docker и WSL полностью остановлены: wsl --list --verbose
→ Попробуй вручную: Optimize-VHD -Path "C:\path\to\docker_data.vhdx" -Mode Full

# Ошибка "Access denied"
→ Запусти PowerShell от имени Администратора
→ Проверь антивирус: может блокировать доступ к VHDX
```

### Исключение из очистки (опционально)

Если нужно сохранить определённые образы:

```powershell
# Перед prune пометить образы
docker tag my-important-image my-important-image:keep
docker system prune -f --filter "label!=keep"
```

---

## 🗂 Чеклист перед запуском

- [ ] Закрыл все приложения, использующие Docker/WSL
- [ ] Запустил PowerShell от имени Администратора
- [ ] Проверил свободное место на `C:` (нужно ~10% от размера VHDX для сжатия)
- [ ] Убедился, что критичные данные сохранены или помечены
- [ ] Протестировал на одном VHDX перед полной очисткой

---

## Ссылки

- 🐳 [Docker Desktop WSL2 Backend](https://docs.docker.com/desktop/wsl/)
- 🔧 [Optimize-VHD Docs](https://learn.microsoft.com/powershell/module/hyper-v/optimize-vhd)
- 🗑️ [docker system prune](https://docs.docker.com/engine/reference/commandline/system_prune/)
- 🔄 [Reclaim disk space in WSL2](https://docs.docker.com/desktop/troubleshoot/topics/#disk-space)
