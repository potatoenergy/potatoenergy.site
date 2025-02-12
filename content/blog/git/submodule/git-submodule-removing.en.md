---
author: ["Potato Energy Team", "ponfertato"]
categories: ["git"]
date: "2023-06-20T15:29:30+03:00"
description: ""
draft: false
series: ["Tips and tricks for working with submodules"]
slug: "removing"
tags: ["git"]
title: "Git Submodule Removing"
---

# About

It often happens that while working on one project, you need to use another project from within it. Perhaps it’s a library that a third party developed or that you’re developing separately and using in multiple parent projects. A common issue arises in these scenarios: you want to be able to treat the two projects as separate yet still be able to use one from within the other.

# Links

[Git - Book](https://git-scm.com/book/en/v2)\
[Git Tools - Submodules](https://git-scm.com/book/en/v2/Git-Tools-Submodules)

# Removing submodule

## Removing a Git Submodule using git submodule deinit

The command `git submodule deinit -f .\themes\hugo-coder` is removing the submodule `hugo-coder` from the repository. It unregisters the submodule and clears the directory associated with it.

```
> git submodule deinit -f .\themes\hugo-coder
Cleared directory 'themes/hugo-coder'
Submodule 'themes/hugo-coder' (https://github.com/luizdepra/hugo-coder.git) unregistered for path 'themes/hugo-coder'
```

## Removing the Directory Associated with a Submodule using rm

The command `rm .\.git\modules\themes -r -fo` is removing the directory associated with the submodule from the `.git/modules` directory. This is necessary to completely remove the submodule from the repository. The `-r` flag is used to remove the directory recursively, and the `-fo` flag is used to force the removal without prompting for confirmation.

```
> rm .\.git\modules\themes -r -fo
```

## Removing a Git Submodule using git rm

The command `git rm -f .\themes\hugo-coder` is removing the submodule `hugo-coder` from the repository. It removes the submodule from the working tree and the index, effectively deleting it from the repository. The `-f` flag is used to force the removal without prompting for confirmation.

```
> git rm -f .\themes\hugo-coder
rm 'themes/hugo-coder'
```

## Checking the Current Status of Your Git Repository with git status

The `git status` command is showing the current status of the Git repository. In this case, it is showing that the branch is `main` and it is up to date with the remote branch `origin/main`. It also shows that there are changes to be committed, specifically the modification of the `.gitmodules` file and the deletion of the `themes/hugo-coder` submodule. The message suggests using the `git restore --staged <file>...` command to unstage the changes if needed.

```
> git status
On branch main
Your branch is up to date with 'origin/main'.

Changes to be committed:
  (use "git restore --staged <file>..." to unstage)
        modified:   .gitmodules
        deleted:    themes/hugo-coder
```

## Committing Changes to Your Git Repository with git commit

The command `git commit -m "Removing submodules"` is committing the changes made to the repository, specifically the removal of the submodule `hugo-coder`. The commit message is "Removing submodules". The output shows that 2 files were changed and 4 deletions were made, including the deletion of the directory associated with the submodule. The line "delete mode 160000 themes/hugo-coder" indicates that the submodule was successfully deleted.

```
> git commit -m "Removing submodules"
[main 72b9422] Removing submodules
 2 files changed, 4 deletions(-)
 delete mode 160000 themes/hugo-coder
```

## Pushing Changes to a Remote Git Repository with git push

The command `git push` is pushing the changes made to the local repository to the remote repository on GitHub. The output shows the progress of the push, including the number of objects being enumerated and counted, the use of delta compression and thread compression, and the writing of objects to the remote repository. The final line shows that the push was successful, with the local `main` branch being updated to match the remote `main` branch.

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
