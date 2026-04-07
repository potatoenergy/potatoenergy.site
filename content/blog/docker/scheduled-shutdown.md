---
author: ["Potato Energy Team", "ponfertato"]
categories: ["docker", "automation", "guide"]
date: "2026-03-17T09:20:00+03:00"
description: "Автоматическая остановка Docker-контейнеров и выключение системы по расписанию. Скрипты, настройка cron, восстановление."
draft: false
series: ["Docker"]
slug: "scheduled-shutdown"
tags: ["docker", "bash", "cron", "automation", "shutdown"]
title: "Docker: Автоматическое выключение по расписанию"
---

Для домашних серверов и тестовых сред актуальна задача экономии ресурсов: ночью или в нерабочее время контейнеры можно останавливать, а систему - выключать. Это руководство описывает безопасный метод с сохранением состояния и автоматическим восстановлением.

> 💡 Метод подходит для OrangePI, Raspberry Pi, старых ПК и любых систем, где важна экономия энергии.

---

## 📦 Скрипт остановки контейнеров и выключения

### Создание скрипта

```bash
# Файл: /usr/local/bin/stop_containers_and_shutdown.sh
cat > /usr/local/bin/stop_containers_and_shutdown.sh << 'EOF'
#!/bin/bash
#
# Скрипт сохраняет ID запущенных контейнеров,
# останавливает их и инициирует завершение работы системы.
#

CONTAINERS_FILE="/etc/active_containers.txt"

echo "=== $(date '+%Y-%m-%d %H:%M:%S') ==="
echo "Запуск скрипта остановки контейнеров и выключения системы"

# Получаем список запущенных контейнеров (по ID)
RUNNING_CONTAINERS=$(docker ps -q)

if [ -n "${RUNNING_CONTAINERS}" ]; then
    echo "Сохранение списка запущенных контейнеров в ${CONTAINERS_FILE}"
    echo "${RUNNING_CONTAINERS}" > "${CONTAINERS_FILE}"
    docker stop ${RUNNING_CONTAINERS}
    echo "Контейнеры остановлены."
else
    echo "Нет запущенных контейнеров."
    [ -f "${CONTAINERS_FILE}" ] && rm -f "${CONTAINERS_FILE}"
fi

sleep 10

echo "Завершение работы системы."
/sbin/shutdown -h now
EOF
```

### Сделать исполняемым

```bash
chmod +x /usr/local/bin/stop_containers_and_shutdown.sh
```

### Как работает

| Шаг                            | Описание                                     |
| ------------------------------ | -------------------------------------------- |
| `docker ps -q`                 | Получает ID всех запущенных контейнеров      |
| `> /etc/active_containers.txt` | Сохраняет список для восстановления          |
| `docker stop`                  | Корректно останавливает контейнеры (SIGTERM) |
| `sleep 10`                     | Даёт время на завершение операций            |
| `shutdown -h now`              | Выключает систему                            |

**Почему так**:

- Сохранение ID позволяет восстановить **те же самые** контейнеры
- `docker stop` даёт контейнерам время на корректное завершение
- `sleep 10` гарантирует, что все данные записаны на диск

---

## 🚀 Скрипт запуска контейнеров

### Создание скрипта

```bash
# Файл: /usr/local/bin/start_containers.sh
cat > /usr/local/bin/start_containers.sh << 'EOF'
#!/bin/bash
#
# Скрипт считывает список контейнеров из файла,
# запускает их и затем удаляет файл.
#

CONTAINERS_FILE="/etc/active_containers.txt"

echo "=== $(date '+%Y-%m-%d %H:%M:%S') ==="
echo "Запуск скрипта старта контейнеров"

if [ ! -f "${CONTAINERS_FILE}" ]; then
    echo "Файл ${CONTAINERS_FILE} не найден. Запуск не требуется."
    exit 0
fi

CONTAINERS=$(cat "${CONTAINERS_FILE}")
if [ -n "${CONTAINERS}" ]; then
    echo "Найден список контейнеров: ${CONTAINERS}"
    for cid in ${CONTAINERS}; do
        docker start "${cid}"
        if [ $? -eq 0 ]; then
            echo "Контейнер ${cid} запущен."
        else
            echo "Ошибка при запуске контейнера ${cid}."
        fi
    done
else
    echo "Файл пуст. Контейнеры запускать не требуется."
fi

rm -f "${CONTAINERS_FILE}"
echo "Файл ${CONTAINERS_FILE} удалён."
EOF
```

### Сделать исполняемым

```bash
chmod +x /usr/local/bin/start_containers.sh
```

### Как работает

| Шаг            | Описание                                           |
| -------------- | -------------------------------------------------- |
| Проверка файла | Если нет - контейнеры не были остановлены скриптом |
| `docker start` | Запускает контейнеры по сохранённым ID             |
| `rm -f`        | Удаляет файл после успешного запуска               |

**Почему удаление файла**:

- Одноразовый маркер - контейнеры не запустятся повторно случайно
- После перезагрузки без скрипта - система работает в обычном режиме

---

## ⏰ Настройка расписания (cron)

### Остановка (ночью)

```bash
# crontab -e (от root)
# Остановка в 00:00 ежедневно
0 0 * * * /usr/local/bin/stop_containers_and_shutdown.sh >> /var/log/docker-shutdown.log 2>&1
```

### Запуск (утром)

```bash
# crontab -e (от root)
# Запуск в 08:00 ежедневно
0 8 * * * /usr/local/bin/start_containers.sh >> /var/log/docker-startup.log 2>&1
```

### Важные замечания

| Параметр          | Описание                            |
| ----------------- | ----------------------------------- |
| `>> /var/log/...` | Логирование для отладки             |
| `2>&1`            | Перенаправление ошибок в тот же лог |
| От root           | Требуется для `docker` и `shutdown` |

**Проверка cron**:

```bash
# Убедиться, что cron запущен
systemctl status cron

# Просмотреть логи
tail -f /var/log/docker-shutdown.log
tail -f /var/log/docker-startup.log
```

---

## 🔧 Альтернатива: systemd timer (современный метод)

### Таймер на остановку

```ini
# /etc/systemd/system/docker-shutdown.timer
[Unit]
Description=Daily Docker shutdown timer

[Timer]
OnCalendar=*-*-* 00:00:00
Persistent=true

[Install]
WantedBy=timers.target
```

### Сервис на остановку

```ini
# /etc/systemd/system/docker-shutdown.service
[Unit]
Description=Stop Docker containers and shutdown
After=docker.service

[Service]
Type=oneshot
ExecStart=/usr/local/bin/stop_containers_and_shutdown.sh
```

### Активация

```bash
systemctl daemon-reload
systemctl enable docker-shutdown.timer
systemctl start docker-shutdown.timer

# Проверить статус
systemctl list-timers | grep docker-shutdown
```

**Преимущества systemd timer**:

- Точное время запуска (не зависит от загрузки cron)
- Встроенное логирование через `journalctl`
- `Persistent=true` - выполнит пропущенный запуск после простоя

---

## 🛡 Безопасность и надёжность

### Исключение критичных контейнеров

Если некоторые контейнеры должны работать постоянно:

```bash
# Модификация скрипта остановки
RUNNING_CONTAINERS=$(docker ps -q --filter "label=shutdown=false")
```

```yaml
# В docker-compose.yml добавить метку
services:
  critical-service:
    image: ...
    labels:
      - "shutdown=false"
```

### Проверка перед выключением

```bash
# Добавить в скрипт остановки
ACTIVE_CONNECTIONS=$(netstat -an | grep ESTABLISHED | wc -l)
if [ "$ACTIVE_CONNECTIONS" -gt 10 ]; then
    echo "Обнаружены активные соединения. Отмена выключения."
    exit 1
fi
```

### Резервное питание (UPS)

```bash
# Интеграция с NUT (Network UPS Tools)
# /etc/nut/upsmon.conf
MONITOR ups@localhost 1 admin password slave
SHUTDOWNCMD "/usr/local/bin/stop_containers_and_shutdown.sh"
```

---

## 📊 Мониторинг и отладка

### Проверка логов

```bash
# Логи скриптов
tail -50 /var/log/docker-shutdown.log
tail -50 /var/log/docker-startup.log

# Логи systemd (если используется)
journalctl -u docker-shutdown.service -n 50
journalctl -u docker-startup.service -n 50
```

### Проверка состояния

```bash
# Какие контейнеры запущены
docker ps --format "table {{.Names}}\t{{.Status}}"

# Статус таймеров
systemctl list-timers | grep docker

# Файл состояния (если существует)
cat /etc/active_containers.txt 2>/dev/null || echo "Файл отсутствует"
```

---

## ⚠️ Частые проблемы

```bash
# Скрипт не выполняется по cron
→ Проверить права: ls -la /usr/local/bin/*.sh
→ Проверить cron: grep CRON /var/log/syslog
→ Убедиться, что cron запущен: systemctl status cron

# Контейнеры не запускаются
→ Проверить Docker: systemctl status docker
→ Проверить логи: docker logs <container_id>
→ Убедиться, что файл существует: cat /etc/active_containers.txt

# Система не выключается
→ Проверить, что shutdown доступен: which shutdown
→ Проверить права: скрипт должен быть от root
→ Посмотреть логи: journalctl -b -1 (предыдущая загрузка)

# Файл не удаляется после запуска
→ Проверить права на файл: ls -la /etc/active_containers.txt
→ Удалить вручную: rm -f /etc/active_containers.txt
```

---

## 🗂 Чеклист перед внедрением

- [ ] Протестировал скрипты вручную перед настройкой cron
- [ ] Проверил логирование: `/var/log/docker-*.log`
- [ ] Убедился, что критичные сервисы исключены (если нужно)
- [ ] Проверил, что данные контейнеров на persistent volumes
- [ ] Настроил уведомления об остановке/запуске (опционально)
- [ ] Создал резервную копию конфигов

---

## Ссылки

- 🐳 [Docker Start/Stop Docs](https://docs.docker.com/engine/reference/commandline/start/)
- ⏰ [Cron Documentation](https://man7.org/linux/man-pages/man5/crontab.5.html)
- 🔧 [Systemd Timer Docs](https://www.freedesktop.org/software/systemd/man/systemd.timer.html)
- 🛡 [NUT - UPS Management](https://networkupstools.org/)
