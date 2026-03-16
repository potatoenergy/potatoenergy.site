---
author: ["Potato Energy Team", "ponfertato"]
categories: ["git", "tutorial"]
date: "2026-03-16T15:25:00+03:00"
description: "Git submodules cheat sheet: add, update, remove. Commands and examples."
draft: false
series: ["Git Tips"]
slug: "submodules"
tags: ["git", "submodules", "workflow"]
title: "Git Submodules: Cheat Sheet"
---

A submodule is a reference to another Git repository inside your project. Useful when you need to include an external library or shared code, but keep it in a separate repo.

> 💡 A submodule stores not the code, but a **reference to a specific commit** of the external repo.

---

## Add a submodule

```bash
# Add a repo as a submodule to a specific path
git submodule add <URL> <path/to/place>

# Example
git submodule add https://github.com/luizdepra/hugo-coder.git themes/hugo-coder

# Commit changes
git commit -m "Add submodule: themes/hugo-coder"
git push
```

After this, your project will have:

- `.gitmodules` file - submodule configuration
- Submodule folder - a reference to the external repo

---

## Clone a project with submodules

```bash
# Option 1: clone with submodules from the start
git clone --recursive <URL>

# Option 2: if you already cloned without submodules
git submodule update --init --recursive
```

---

## Update a submodule

```bash
# Go into the submodule and pull changes
cd themes/hugo-coder
git pull origin main

# Go back to root and commit the new submodule commit
cd ../..
git add themes/hugo-coder
git commit -m "Update submodule: hugo-coder"
git push
```

**Or one command from root:**

```bash
# Update all submodules to latest remote commits
git submodule update --init --recursive --remote

# Commit the updated references
git add .
git commit -m "Update all submodules"
git push
```

> ⚠️ `--remote` fetches latest commits from remote repos. Without it, only commits already recorded in `.gitmodules` are used.

---

## Remove a submodule

```bash
# 1. Deinitialize the submodule
git submodule deinit -f themes/hugo-coder

# 2. Remove from index and working directory
git rm -f themes/hugo-coder

# 3. Remove internal data (optional but recommended)
rm -rf .git/modules/themes/hugo-coder

# 4. Commit changes
git commit -m "Remove submodule: themes/hugo-coder"
git push
```

---

## Useful commands

```bash
# Show status of all submodules
git submodule status

# Show which commits are pending update
git submodule foreach 'git log -1 --oneline'

# Sync submodule URLs (if remote changed)
git submodule sync --recursive

# Run a command in all submodules
git submodule foreach 'git fetch'
```

---

## Troubleshooting

```bash
# Submodule folder is empty after clone
→ git submodule update --init --recursive

# "fatal: not a git repository" inside submodule
→ Delete the submodule folder and run:
  git submodule update --init

# Submodule version conflict during merge
→ Choose the correct commit version:
  git add themes/hugo-coder
  git commit -m "Resolve submodule conflict"
```

---

## Links

- 📘 [Official Git Book: Submodules](https://git-scm.com/book/en/v2/Git-Tools-Submodules)
- 🐙 [git-submodule docs](https://git-scm.com/docs/git-submodule)
