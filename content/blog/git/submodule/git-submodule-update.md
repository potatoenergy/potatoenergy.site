---
author: ["Potato Energy Team", "ponfertato"]
categories: ["git"]
date: "2023-06-20T15:30:00+03:00"
description: ""
draft: false
series: ["Советы и рекомендации по работе с подмодулями"]
slug: "update"
tags: ["git"]
title: "Обновление подмодуля Git"
---

# Об

Часто бывает так, что во время работы над одним проектом вам необходимо использовать другой проект из его состава. Возможно, это библиотека, разработанная третьей стороной, или библиотека, которую вы разрабатываете отдельно и используете в нескольких родительских проектах. В таких случаях возникает общая проблема: вы хотите иметь возможность рассматривать эти два проекта как отдельные, но при этом иметь возможность использовать один из них из другого.

# Ссылки

[Git - Книга](https://git-scm.com/book/ru/v2)\
[Инструменты Git - Подмодули](https://git-scm.com/book/ru/v2/Инструменты-Git-Подмодули)

# Обновление подмодуля

## Обновление субмодулей Git с помощью git submodule update

Команда `git submodule update --init --recursive --remote` обновляет подмодуль Git в локальном хранилище. Она инициализирует подмодуль, если он не был инициализирован ранее, рекурсивно обновляет все вложенные подмодули и получает последние изменения из удаленного хранилища. Вывод показывает ход обновления, включая перечисление и подсчет объектов, повторное использование любых существующих объектов и распаковку новых объектов. Он также показывает хэш-диапазон фиксации обновленного субмодуля.

```
> git submodule update --init --recursive --remote
remote: Enumerating objects: 4, done.
remote: Counting objects: 100% (4/4), done.
remote: Total 4 (delta 3), reused 4 (delta 3), pack-reused 0
Unpacking objects: 100% (4/4), 1.29 KiB | 44.00 KiB/s, done.
From https://github.com/luizdepra/hugo-coder
   86ed09e..e0969a4  main       -> origin/main
Submodule path 'themes/hugo-coder': checked out 'e0969a4ab96d939527a31101764b8bf780788dd9'
```

## Проверка текущего состояния вашего Git-репозитория с помощью git status

Команда `git status` показывает текущее состояние репозитория. В данном случае она показывает, что подмодуль `themes/hugo-coder` был изменен и готов к фиксации.

```
> git status
On branch main
Your branch is up to date with 'origin/main'.

Changes to be committed:
  (use "git restore --staged <file>..." to unstage)
        modified:   themes/hugo-coder
```

## Фиксация изменений в вашем Git-репозитории с помощью git commit

Команда `git commit -m "Update submodules"` фиксирует изменения, внесенные в подмодуль, в локальном репозитории с сообщением о фиксации "Update submodules". Вывод `[main 654c4df] Update submodules 1 file changed, 1 insertion(+), 1 deletion(-)` указывает, что фиксация прошла успешно и что один файл был изменён одной вставкой и одним удалением.

```
> git commit -m "Update submodules"
[main 654c4df] Update submodules
 1 file changed, 1 insertion(+), 1 deletion(-)
```

## Передача изменений в удаленный Git-репозиторий с помощью git push

Команда `git push` продвигает зафиксированные изменения в удаленный репозиторий. В данном случае она перемещает изменения, внесенные в подмодуль, в ветвь `main` удаленного хранилища `https://github.com/ponfertato/example.git`. Вывод показывает ход выполнения push, включая количество перечисляемых и подсчитываемых объектов, использование дельта-сжатия и потоков, а также разрешение любых дельт. Последняя строка показывает успешный push с обновленным хэшем коммита `64c8ca1..654c4df`.

```
> git push
Enumerating objects: 5, done.
Counting objects: 100% (5/5), done.
Delta compression using up to 4 threads
Compressing objects: 100% (2/2), done.
Writing objects: 100% (3/3), 288 bytes | 288.00 KiB/s, done.
Total 3 (delta 1), reused 0 (delta 0), pack-reused 0
remote: Resolving deltas: 100% (1/1), completed with 1 local object.
To https://github.com/ponfertato/example.git
   64c8ca1..654c4df  main -> main
```
