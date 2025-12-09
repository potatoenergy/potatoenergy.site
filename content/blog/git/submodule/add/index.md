---
author: ["Potato Energy Team", "ponfertato"]
categories: ["git"]
date: "2023-06-20T15:29:00+03:00"
description: ""
draft: false
series: ["Советы и рекомендации по работе с подмодулями"]
tags: ["git"]
title: "Добавление подмодуля Git"
---

# Об

Часто бывает так, что во время работы над одним проектом вам необходимо использовать другой проект из его состава. Возможно, это библиотека, разработанная третьей стороной, или библиотека, которую вы разрабатываете отдельно и используете в нескольких родительских проектах. В таких случаях возникает общая проблема: вы хотите иметь возможность рассматривать эти два проекта как отдельные, но при этом иметь возможность использовать один из них из другого.

# Ссылки

[Git - Книга](https://git-scm.com/book/ru/v2)\
[Инструменты Git - Подмодули](https://git-scm.com/book/ru/v2/Инструменты-Git-Подмодули)

# Добавление подмодуля

## Добавление нового подмодуля в ваш Git-репозиторий с помощью git submodule add

Эта команда добавляет новый подмодуль в текущий Git-репозиторий. Она клонирует репозиторий, расположенный по адресу <https://github.com/luizdepra/hugo-coder.git>, и добавляет его в качестве подмодуля в каталог themes/hugo-coder текущего репозитория. Вывод показывает прогресс процесса клонирования и любые предупреждения или ошибки, которые могут возникнуть.

```
> git submodule add https://github.com/luizdepra/hugo-coder.git .\themes\hugo-coder
Cloning into 'C:/Users/ponfertato/Documents/Project/Git/test/themes/hugo-coder'...
remote: Enumerating objects: 3547, done.
remote: Counting objects: 100% (112/112), done.
remote: Compressing objects: 100% (60/60), done.
remote: Total 3547 (delta 46), reused 89 (delta 40), pack-reused 3435
Receiving objects: 100% (3547/3547), 3.12 MiB | 6.96 MiB/s, done.
Resolving deltas: 100% (1840/1840), done.
warning: in the working copy of '.gitmodules', LF will be replaced by CRLF the next time Git touches it
```

## Проверка текущего состояния вашего Git-репозитория с помощью git status

Команда `git status` показывает текущее состояние репозитория Git. В данном случае она показывает, что есть новые файлы, которые должны быть зафиксированы, а именно файл `.gitmodules` и каталог `themes/hugo-coder`, которые были добавлены в качестве подмодуля в предыдущем шаге. В сообщении также содержится предложение о том, как снять фиксацию изменений, если это необходимо.

```
> git status
On branch main

No commits yet

Changes to be committed:
  (use "git rm --cached <file>..." to unstage)
        new file:   .gitmodules
        new file:   themes/hugo-coder
```

## Фиксация изменений в вашем Git-репозитории с помощью git commit

Команда `git commit -m "Adding submodules"` фиксирует изменения, внесенные в Git-репозиторий, а именно добавление нового подмодуля. Флаг `-m` используется для добавления сообщения о фиксации, которое в данном случае звучит как "Добавление субмодулей". Вывод показывает хэш фиксации (`464658f`) и детали сделанных изменений, включая создание файла `.gitmodules` и каталога `themes/hugo-coder` в качестве подмодуля.

```
> git commit -m "Adding submodules"
[main (root-commit) 464658f] Adding submodules
 2 files changed, 4 insertions(+)
 create mode 100644 .gitmodules
 create mode 160000 themes/hugo-coder
```

## Передача изменений в удаленный Git-репозиторий с помощью git push

Команда `git push` перемещает зафиксированные изменения в удаленный репозиторий, в данном случае в ветку `main` репозитория `https://github.com/ponfertato/example.git`. Вывод показывает прогресс процесса push, включая количество перечисляемых и подсчитываемых объектов, использование дельта-сжатия и сжатия объектов, а также общий размер записанных объектов. Последняя строка показывает, что в удаленном хранилище была создана новая ветвь под названием `main`.

```
> git push
Enumerating objects: 4, done.
Counting objects: 100% (4/4), done.
Delta compression using up to 4 threads
Compressing objects: 100% (3/3), done.
Writing objects: 100% (4/4), 372 bytes | 372.00 KiB/s, done.
Total 4 (delta 0), reused 0 (delta 0), pack-reused 0
To https://github.com/ponfertato/example.git
 * [new branch]      main -> main
```
