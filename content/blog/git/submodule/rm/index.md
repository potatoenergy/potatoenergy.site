---
author: ["Potato Energy Team", "ponfertato"]
categories: ["git"]
date: "2023-06-20T15:29:30+03:00"
description: ""
draft: false
series: ["Советы и рекомендации по работе с подмодулями"]
tags: ["git"]
title: "Удаление подмодуля Git"
---

# Об

Часто бывает так, что во время работы над одним проектом вам необходимо использовать другой проект из его состава. Возможно, это библиотека, разработанная третьей стороной, или библиотека, которую вы разрабатываете отдельно и используете в нескольких родительских проектах. В таких случаях возникает общая проблема: вы хотите иметь возможность рассматривать эти два проекта как отдельные, но при этом иметь возможность использовать один из них из другого.

# Ссылки

[Git - Книга](https://git-scm.com/book/ru/v2)\
[Инструменты Git - Подмодули](https://git-scm.com/book/ru/v2/Инструменты-Git-Подмодули)

# Удаление подмодуля

## Удаление Git-подмодуля с помощью git submodule deinit

Команда `git submodule deinit -f .\themes\hugo-coder` удаляет субмодуль `hugo-coder` из репозитория. Она снимает субмодуль с регистрации и очищает связанный с ним каталог.

```
> git submodule deinit -f .\themes\hugo-coder
Cleared directory 'themes/hugo-coder'
Submodule 'themes/hugo-coder' (https://github.com/luizdepra/hugo-coder.git) unregistered for path 'themes/hugo-coder'
```

## Удаление каталога, связанного с подмодулем, с помощью rm

Команда `rm .\.git\modules\themes -r -fo` удаляет каталог, связанный с подмодулем, из каталога `.git/modules`. Это необходимо для полного удаления подмодуля из репозитория. Флаг `-r` используется для рекурсивного удаления каталога, а флаг `-fo` - для принудительного удаления без запроса подтверждения.

```
> rm .\.git\modules\themes -r -fo
```

## Удаление Git-подмодуля с помощью git rm

Команда `git rm -f .\themes\hugo-coder` удаляет подмодуль `hugo-coder` из репозитория. Она удаляет подмодуль из рабочего дерева и индекса, фактически удаляя его из хранилища. Флаг `-f` используется для принудительного удаления без запроса подтверждения.

```
> git rm -f .\themes\hugo-coder
rm 'themes/hugo-coder'
```

## Проверка текущего состояния вашего Git-репозитория с помощью git status

Команда `git status` показывает текущее состояние Git-репозитория. В данном случае она показывает, что ветка `main` и она обновляется с удаленной веткой `origin/main`. Он также показывает, что есть изменения, которые должны быть зафиксированы, а именно изменение файла `.gitmodules` и удаление подмодуля `themes/hugo-coder`. В сообщении предлагается использовать команду `git restore --staged <file>...`, чтобы снять фиксацию изменений, если это необходимо.

```
> git status
On branch main
Your branch is up to date with 'origin/main'.

Changes to be committed:
  (use "git restore --staged <file>..." to unstage)
        modified:   .gitmodules
        deleted:    themes/hugo-coder
```

## Фиксация изменений в вашем Git-репозитории с помощью git commit

Команда `git commit -m "Removing submodules"` фиксирует изменения, внесенные в репозиторий, а именно удаление субмодуля `hugo-coder`. Сообщение о фиксации - "Удаление подмодулей". Вывод показывает, что 2 файла были изменены и 4 удалены, включая удаление каталога, связанного с подмодулем. Строка "delete mode 160000 themes/hugo-coder" указывает на то, что подмодуль был успешно удален.

```
> git commit -m "Removing submodules"
[main 72b9422] Removing submodules
 2 files changed, 4 deletions(-)
 delete mode 160000 themes/hugo-coder
```

## Передача изменений в удаленный Git-репозиторий с помощью git push

Команда `git push` перемещает изменения, внесенные в локальный репозиторий, в удаленный репозиторий на GitHub. Вывод показывает ход выполнения push, включая количество перечисляемых и подсчитываемых объектов, использование дельта-сжатия и потокового сжатия, а также запись объектов в удаленный репозиторий. Последняя строка показывает, что push был успешным, локальная ветка `main` была обновлена, чтобы соответствовать удаленной ветке `main`.

```
> git push
Enumerating objects: 5, done.
Counting objects: 100% (5/5), done.
Delta compression using up to 4 threads
Compressing objects: 100% (1/1), done.
Writing objects: 100% (3/3), 243 bytes | 243.00 KiB/s, done.
Total 3 (delta 0), reused 0 (delta 0), pack-reused 0
To https://github.com/ponfertato/example.git
   a79a63f..72b9422  main -> main
```
