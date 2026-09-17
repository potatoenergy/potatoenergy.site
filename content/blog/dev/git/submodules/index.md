---
author: ["Potato Energy Team", "ponfertato"]
categories: ["git", "tutorial"]
date: "2026-03-16T15:25:00+03:00"
description: "Шпаргалка по подмодулям Git: добавление, обновление, удаление. Команды и примеры."
draft: false
series: ["Git Tips"]
slug: "submodules"
tags: ["git", "submodules", "workflow"]
title: "Git Submodules: Шпаргалка"
aliases:
  - /blog/git/submodules/
---

Подмодуль - ссылка на другой Git-репозиторий внутри проекта. Используется для подключения внешней библиотеки или общего кода, хранящегося в отдельном репозитории.

> 💡 Подмодуль хранит ссылку на конкретный коммит внешнего репозитория, а не код.

---

## Добавить подмодуль

```bash
# Добавить репозиторий как подмодуль в указанную папку
git submodule add <URL> <путь/куда/положить>

# Пример
git submodule add https://github.com/luizdepra/hugo-coder.git themes/hugo-coder

# Зафиксировать изменения
git commit -m "Add submodule: themes/hugo-coder"
git push
```

После этого в проекте появятся:

- Файл `.gitmodules` - конфигурация подмодулей
- Папка с подмодулем - ссылка на внешний репозиторий

---

## Клонировать проект с подмодулями

```bash
# Вариант 1: сразу с подмодулями
git clone --recursive <URL>

# Вариант 2: если уже склонировали без подмодулей
git submodule update --init --recursive
```

---

## Обновить подмодуль

```bash
# Зайти в подмодуль и подтянуть изменения
cd themes/hugo-coder
git pull origin main

# Вернуться в корень и зафиксировать новый коммит подмодуля
cd ../..
git add themes/hugo-coder
git commit -m "Update submodule: hugo-coder"
git push
```

**Или одной командой из корня:**

```bash
# Обновить все подмодули до последних коммитов удалённых веток
git submodule update --init --recursive --remote

# Зафиксировать изменения ссылок
git add .
git commit -m "Update all submodules"
git push
```

> ⚠️ `--remote` тянет последние коммиты из удалённых репозиториев. Без него - только те коммиты, что уже зафиксированы в `.gitmodules`.

---

## Удалить подмодуль

```bash
# 1. Деинициализировать подмодуль
git submodule deinit -f themes/hugo-coder

# 2. Удалить из индекса и рабочей директории
git rm -f themes/hugo-coder

# 3. Удалить служебные данные (опционально, но рекомендуется)
rm -rf .git/modules/themes/hugo-coder

# 4. Зафиксировать изменения
git commit -m "Remove submodule: themes/hugo-coder"
git push
```

---

## Полезные команды

```bash
# Показать статус всех подмодулей
git submodule status

# Показать, какие коммиты ждут обновления
git submodule foreach 'git log -1 --oneline'

# Синхронизировать URL подмодулей (если изменился remote)
git submodule sync --recursive

# Выполнить команду во всех подмодулях
git submodule foreach 'git fetch'
```

---

## Типичные проблемы

```bash
# Подмодуль пуст после клонирования
→ git submodule update --init --recursive

# Ошибка "fatal: not a git repository" внутри подмодуля
→ Удалить папку подмодуля и выполнить:
  git submodule update --init

# Конфликт версий подмодуля при мердже
→ Выбрать нужную версию коммита:
  git add themes/hugo-coder
  git commit -m "Resolve submodule conflict"
```

---

## Ссылки

- [Официальная книга Git: Подмодули](https://git-scm.com/book/ru/v2/Инструменты-Git-Подмодули)
- [Документация git-submodule](https://git-scm.com/docs/git-submodule)
