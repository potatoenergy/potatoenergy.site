---
author: ["Potato Energy Team", "ponfertato"]
categories: ["git", "tutorial", "setup"]
date: "2026-03-16T20:00:00+03:00"
description: "Быстрая установка и настройка Git на Windows и Linux. Базовая конфигурация, SSH-ключи, первые шаги."
draft: false
series: ["Git"]
slug: "install"
tags: ["git", "install", "setup", "ssh", "windows", "linux"]
title: "Git: Установка и настройка"
---

Git - система контроля версий. Сохраняет историю изменений кода, позволяет работать в команде и откатывать ошибки.

> 💡 Установил один раз - пользуешься годами.

---

## Установка

### Windows

**Способ 1: Официальный установщик (рекомендуется)**

1. Скачай с [git-scm.com](https://git-scm.com/download/win)
2. Запусти установщик, оставляй настройки по умолчанию
3. **Важно**: на шаге "Adjusting your PATH" выбери `Git from the command line and also from 3rd-party software`

**Способ 2: Через Winget (PowerShell)**

```powershell
winget install Git.Git
```

**Способ 3: Через Chocolatey**

```powershell
choco install git -y
```

### Linux

**Ubuntu / Debian**

```bash
sudo apt update
sudo apt install git -y
```

**Fedora / RHEL**

```bash
sudo dnf install git -y
```

**Arch / Manjaro**

```bash
sudo pacman -S git
```

**Проверка установки (все системы)**

```bash
git --version
# Ожидаемый вывод: git version 2.x.x
```

---

## Базовая настройка

```bash
# Имя и почта (будут в каждом коммите)
git config --global user.name "Kirill Ponfertato"
git config --global user.email "kirill@potatoenergy.ru"

# Редактор по умолчанию (выбери один)
git config --global core.editor "code --wait"      # VS Code
git config --global core.editor "nano"             # Nano
git config --global core.editor "vim"              # Vim

# Имя ветки по умолчанию
git config --global init.defaultBranch main

# Цветной вывод
git config --global color.ui auto

# Авто-CRLF для Windows (перевод строк)
git config --global core.autocrlf true    # Windows
git config --global core.autocrlf input   # Linux/macOS

# Просмотр всех настроек
git config --global --list
```

---

## SSH-ключи (для GitHub / GitLab)

```bash
# Создать ключ (нажми Enter на все вопросы)
ssh-keygen -t ed25519 -C "kirill@potatoenergy.ru"

# Если ed25519 не поддерживается:
ssh-keygen -t rsa -b 4096 -C "kirill@potatoenergy.ru"

# Запустить SSH-агент и добавить ключ
eval "$(ssh-agent -s)"
ssh-add ~/.ssh/id_ed25519

# Показать публичный ключ (скопировать содержимое)
cat ~/.ssh/id_ed25519.pub
# Windows (PowerShell):
Get-Content ~/.ssh/id_ed25519.pub

# Добавить ключ в:
# • GitHub: Settings → SSH and GPG keys → New SSH key
# • GitLab: User Settings → SSH Keys
```

**Проверка подключения**

```bash
# GitHub
ssh -T git@github.com
# Ожидаемо: "Hi ponfertato! You've successfully authenticated"

# GitLab
ssh -T git@gitlab.com
```

---

## Первые шаги

```bash
# Создать новый репозиторий
mkdir my-project && cd my-project
git init

# Клонировать существующий
git clone <URL>
git clone <URL> my-folder          # в папку с именем
git clone --depth 1 <URL>          # только последний коммит

# Проверить статус
git status

# Добавить файлы и сделать коммит
git add .
git commit -m "Initial commit"

# Подключить удалённый репозиторий
git remote add origin git@github.com:ponfertato/my-project.git
git push -u origin main
```

---

## Полезные алиасы (опционально)

```bash
# Добавить в ~/.gitconfig или выполнить команды:
git config --global alias.co checkout
git config --global alias.br branch
git config --global alias.ci commit
git config --global alias.st "status -s"
git config --global alias.lg "log --oneline --graph --all"

# Использование:
git st      # вместо git status -s
git lg      # красивый лог
git co -b feature  # создать и переключиться на ветку
```

---

## Если что-то не так

```bash
# Git не найден после установки
→ Перезапусти терминал / компьютер

# Ошибка "Permission denied (publickey)"
→ Проверь, что публичный ключ добавлен в профиль GitHub/GitLab
→ Проверь: ssh -T git@github.com

# Конфликт переводов строк (CRLF/LF)
→ Настрой core.autocrlf под свою ОС (см. выше)

# Сбросить все настройки к дефолтным
git config --global --unset-all user.name
git config --global --unset-all user.email
# Или удалить файл: ~/.gitconfig
```

---

## Ссылки

- 🌐 [Официальный сайт](https://git-scm.com)
- 📘 [Книга Pro Git (бесплатно)](https://git-scm.com/book/ru/v2)
- 🎮 [Интерактивный тренажёр](https://learngitbranching.js.org/?locale=ru_RU)
- 🔑 [Генератор .gitignore](https://gitignore.io)
