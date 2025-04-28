---
author: ["Potato Energy Team", "ponfertato"]
categories: ["git"]
date: "2023-06-20T15:30:00+03:00"
description: ""
draft: false
series: ["Tips and tricks for working with submodules"]
tags: ["git"]
title: "Git Submodule Update"
---

# About

It often happens that while working on one project, you need to use another project from within it. Perhaps it’s a library that a third party developed or that you’re developing separately and using in multiple parent projects. A common issue arises in these scenarios: you want to be able to treat the two projects as separate yet still be able to use one from within the other.

# Links

[Git - Book](https://git-scm.com/book/en/v2)\
[Git Tools - Submodules](https://git-scm.com/book/en/v2/Git-Tools-Submodules)

# Updating submodule

## Updating Git submodules using git submodule update

The command `git submodule update --init --recursive --remote` is updating the Git submodule in the local repository. It is initializing the submodule if it has not been initialized before, recursively updating any nested submodules, and fetching the latest changes from the remote repository. The output shows the progress of the update, including the enumeration and counting of objects, the reuse of any existing objects, and the unpacking of new objects. It also shows the commit hash range of the updated submodule.

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

## Checking the Current Status of Your Git Repository with git status

The command `git status` is showing the current status of the repository. In this case, it is showing that the submodule `themes/hugo-coder` has been modified and is ready to be committed.

```
> git status
On branch main
Your branch is up to date with 'origin/main'.

Changes to be committed:
  (use "git restore --staged <file>..." to unstage)
        modified:   themes/hugo-coder
```

## Committing Changes to Your Git Repository with git commit

The command `git commit -m "Update submodules"` is committing the changes made to the submodule to the local repository with the commit message "Update submodules". The output `[main 654c4df] Update submodules 1 file changed, 1 insertion(+), 1 deletion(-)` indicates that the commit was successful and that one file was changed with one insertion and one deletion.

```
> git commit -m "Update submodules"
[main 654c4df] Update submodules
 1 file changed, 1 insertion(+), 1 deletion(-)
```

## Pushing Changes to a Remote Git Repository with git push

The command `git push` is pushing the committed changes to the remote repository. In this case, it is pushing the changes made to the submodule to the `main` branch of the remote repository `https://github.com/ponfertato/example.git`. The output shows the progress of the push, including the number of objects being enumerated and counted, the use of delta compression and threads, and the resolution of any deltas. The final line shows the successful push with the updated commit hash range `64c8ca1..654c4df`.

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
