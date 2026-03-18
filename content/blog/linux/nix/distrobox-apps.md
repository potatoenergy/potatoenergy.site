---
author: ["Potato Energy Team", "ponfertato"]
categories: ["nix", "linux", "containers"]
date: "2026-03-18T17:25:00+03:00"
description: "Запуск программ для Ubuntu/Debian на NixOS через Distrobox. Контейнер, установка, ярлык, полная очистка."
draft: false
series: ["Nix/NixOS"]
slug: "distrobox-apps"
tags: ["nix", "distrobox", "containers", "desktop"]
title: "NixOS: Запуск «чужих» программ через Distrobox"
---

Иногда на NixOS нужно запустить программу, которая есть только для Ubuntu/Debian или распространяется в виде `.deb`/`.iso`. Distrobox позволяет запустить её в контейнере, но с интеграцией в систему хоста - как нативное приложение.

> 💡 Метод работает на любом дистрибутиве с установленным Nix.

---

## 📦 Создание контейнера

```bash
# Создать контейнер с пробросом нужных директорий
nix shell nixpkgs#distrobox --command distrobox create \
  --image ubuntu:20.04 \
  --name <container-name> \
  --volume "<volume_pts>:/dev/pts" \
  --volume "<volume_journal>:/var/log/journal" \
  --home "$HOME/<container-name>-home"
```

**Параметры**:

- `--image` - базовый образ (ubuntu:20.04, debian:11, etc.)
- `--volume` - проброс директорий (аудио, логи)
- `--home` - отдельная домашняя директория внутри контейнера
- `--name` - имя контейнера (используется в других командах)

**Войти в контейнер**:

```bash
nix shell nixpkgs#distrobox --command distrobox enter <container-name>
```

---

## 🔧 Установка программы внутри контейнера

```bash
# Внутри контейнера (после distrobox enter)
sudo sh -c '
  # Очистка старых конфигов
  rm -rf /etc/apt/sources.list.d/<vendor>.list*
  umount /mnt/<vendor> 2>/dev/null || true

  # Скачивание и монтирование образа
  wget -O /tmp/<vendor>.iso "https://<url>/<vendor>.iso"
  mkdir -p /mnt/<vendor>
  mount -o loop,ro -t iso9660 /tmp/<vendor>.iso /mnt/<vendor>

  # Установка из локального репозитория
  echo "deb [trusted=yes] file:/mnt/<vendor>/repo ./" > /etc/apt/sources.list.d/<vendor>.list
  apt update
  apt install -y <package-name>-full

  # Уборка
  umount /mnt/<vendor>
  rm -f /etc/apt/sources.list.d/<vendor>.list /tmp/<vendor>.iso
'
```

**Заменить плейсхолдеры**:
| `<vendor>` | Имя вендора (например, `nausoftphone`) |
| `<package-name>` | Имя пакета в репозитории |
| `<url>` | Ссылка на установочный образ |

**Проверить**:

```bash
<package-name> --version
```

---

## 🖥 Ярлык для запуска (.desktop)

```desktop
# ~/.local/share/applications/<app>-distrobox.desktop
[Desktop Entry]
Version=1.0
Type=Application
Name=<App Name> (Distrobox)
Comment=Run <App Name> inside Distrobox container
Exec=konsole --hold -e bash -c 'nix shell nixpkgs#distrobox --command distrobox enter --name <container-name> -- <app-command>'
Icon=<icon-name>
Terminal=false
Categories=Network;AudioVideo;
Keywords=<app>;distrobox;container;
```

**Применить**:

```bash
update-desktop-database ~/.local/share/applications/
```

Теперь приложение доступно в меню, лаунчерах и поиске.

---

## 🔄 Обновление программы

```bash
# Войти в контейнер
nix shell nixpkgs#distrobox --command distrobox enter <container-name>

# Обновить пакет
sudo apt update && sudo apt install --only-upgrade <package-name>-full
```

---

## 🗑 Полное удаление (контейнер + volumes)

```bash
# 1. Удалить контейнер
distrobox rm <container-name>

# 2. Удалить привязанные volumes (если создавались отдельно)
# Для Podman:
podman volume rm <volume_pts> <volume_journal>

# Для Docker:
docker volume rm <volume_pts> <volume_journal>

# 3. Удалить домашнюю директорию контейнера (если использовалась)
rm -rf "$HOME/<container-name>-home"

# 4. Удалить ярлык
rm -f ~/.local/share/applications/<app>-distrobox.desktop
update-desktop-database ~/.local/share/applications/
```

> ⚠️ Удаление volumes необратимо - убедитесь, что внутри нет нужных данных.

**Проверить, что всё удалено**:

```bash
# Контейнеры
distrobox list | grep <container-name>

# Volumes (Podman)
podman volume ls | grep <volume_pts>

# Volumes (Docker)
docker volume ls | grep <volume_pts>
```

---

## ⚙️ Оптимизация запуска

### Скрыть терминал (для готовых решений)

```desktop
# Заменить Exec на:
Exec=sh -c 'nix shell nixpkgs#distrobox --command distrobox enter --name <container-name> -- <app-command>' >> ~/.cache/<app>.log 2>&1 &
```

### Проброс аудио (если не работает)

```bash
# Добавить при создании контейнера:
--volume "$XDG_RUNTIME_DIR/pulse:/run/user/1000/pulse"
# Или для PipeWire:
--volume "$XDG_RUNTIME_DIR/pipewire-0:/run/user/1000/pipewire-0"
```

---

## ⚠️ Частые проблемы

```bash
# Ярлык не появляется
→ Проверить синтаксис: desktop-file-validate ~/.local/share/applications/<app>-distrobox.desktop
→ Обновить кэш: update-desktop-database ~/.local/share/applications/

# Контейнер не запускается после обновления Nix
→ Пересоздать: distrobox rm <name> && создать заново
→ Данные в --home сохранятся, если не удалять директорию вручную

# Нет доступа к аудио/устройствам
→ Проверить проброс volumes при создании контейнера
→ Убедиться, что пользователь в нужных группах (audio, input)

# Volume не удаляется ("in use")
→ Убедиться, что контейнер остановлен: distrobox list
→ Остановить принудительно: podman stop <container-name> || docker stop <container-name>
```

---

## Ссылки

- 🐧 [Distrobox Docs](https://distrobox.it/)
- 🦊 [NixOS Wiki: Distrobox](https://wiki.nixos.org/wiki/Distrobox)
- 🖥 [Desktop Entry Spec](https://specifications.freedesktop.org/desktop-entry-spec/)
