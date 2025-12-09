---
author: ["Potato Energy Team", "ponfertato"]
categories: ["git"]
date: "2023-06-20T15:29:00+03:00"
description: ""
draft: false
series: ["Tips and tricks for working with submodules"]
tags: ["git"]
title: "Git Submodule Adding"
---

# About

It often happens that while working on one project, you need to use another project from within it. Perhaps it’s a library that a third party developed or that you’re developing separately and using in multiple parent projects. A common issue arises in these scenarios: you want to be able to treat the two projects as separate yet still be able to use one from within the other.

# Links

[Git - Book](https://git-scm.com/book/en/v2)\
[Git Tools - Submodules](https://git-scm.com/book/en/v2/Git-Tools-Submodules)

# Adding submodule

## Adding a New Submodule to Your Git Repository with git submodule add

This command is adding a new submodule to the current Git repository. It is cloning the repository located at <https://github.com/luizdepra/hugo-coder.git> and adding it as a submodule in the themes/hugo-coder directory of the current repository. The output shows the progress of the cloning process and any warnings or errors that may occur.

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

## Checking the Current Status of Your Git Repository with git status

The `git status` command is showing the current status of the Git repository. In this case, it is showing that there are new files to be committed, specifically the `.gitmodules` file and the `themes/hugo-coder` directory, which were added as a submodule in the previous step. The message also provides a suggestion on how to unstage the changes if needed.

```
> git status
On branch main

No commits yet

Changes to be committed:
  (use "git rm --cached <file>..." to unstage)
        new file:   .gitmodules
        new file:   themes/hugo-coder
```

## Committing Changes to Your Git Repository with git commit

The `git commit -m "Adding submodules"` command is committing the changes made to the Git repository, specifically the addition of a new submodule. The `-m` flag is used to add a commit message, which in this case is "Adding submodules". The output shows the commit hash (`464658f`) and the details of the changes made, including the creation of the `.gitmodules` file and the `themes/hugo-coder` directory as a submodule.

```
> git commit -m "Adding submodules"
[main (root-commit) 464658f] Adding submodules
 2 files changed, 4 insertions(+)
 create mode 100644 .gitmodules
 create mode 160000 themes/hugo-coder
```

## Pushing Changes to a Remote Git Repository with git push

The `git push` command is pushing the committed changes to the remote repository, in this case, the `main` branch of the `https://github.com/ponfertato/example.git` repository. The output shows the progress of the push process, including the number of objects enumerated and counted, the use of delta compression and object compression, and the total size of the objects written. The final line shows that a new branch called `main` was created in the remote repository.

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
