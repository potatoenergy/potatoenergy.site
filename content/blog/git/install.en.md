---
author: ["Potato Energy Team", "ponfertato"]
categories: ["git", "tutorial", "setup"]
date: "2026-03-16T15:25:00+03:00"
description: "Quick Git installation and setup on Windows and Linux. Basic config, SSH keys, first steps."
draft: false
series: ["Git"]
slug: "install"
tags: ["git", "install", "setup", "ssh", "windows", "linux"]
title: "Git: Installation & Setup"
---

Git is a version control system. It tracks code changes, enables team collaboration, and lets you undo mistakes.

> 💡 Install once, use for years.

---

## Installation

### Windows

**Option 1: Official installer (recommended)**

1. Download from [git-scm.com](https://git-scm.com/download/win)
2. Run installer, keep defaults
3. **Important**: On "Adjusting your PATH" step, select `Git from the command line and also from 3rd-party software`

**Option 2: Via Winget (PowerShell)**

```powershell
winget install Git.Git
```

**Option 3: Via Chocolatey**

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

**Verify installation (all systems)**

```bash
git --version
# Expected: git version 2.x.x
```

---

## Basic configuration

```bash
# Name and email (appear in every commit)
git config --global user.name "Kirill Ponfertato"
git config --global user.email "kirill@potatoenergy.ru"

# Default editor (pick one)
git config --global core.editor "code --wait"      # VS Code
git config --global core.editor "nano"             # Nano
git config --global core.editor "vim"              # Vim

# Default branch name
git config --global init.defaultBranch main

# Colored output
git config --global color.ui auto

# Auto CRLF for line endings
git config --global core.autocrlf true    # Windows
git config --global core.autocrlf input   # Linux/macOS

# View all settings
git config --global --list
```

---

## SSH keys (for GitHub / GitLab)

```bash
# Generate key (press Enter for all prompts)
ssh-keygen -t ed25519 -C "kirill@potatoenergy.ru"

# If ed25519 not supported:
ssh-keygen -t rsa -b 4096 -C "kirill@potatoenergy.ru"

# Start SSH agent and add key
eval "$(ssh-agent -s)"
ssh-add ~/.ssh/id_ed25519

# Show public key (copy the output)
cat ~/.ssh/id_ed25519.pub
# Windows (PowerShell):
Get-Content ~/.ssh/id_ed25519.pub

# Add key to:
# • GitHub: Settings → SSH and GPG keys → New SSH key
# • GitLab: User Settings → SSH Keys
```

**Test connection**

```bash
# GitHub
ssh -T git@github.com
# Expected: "Hi ponfertato! You've successfully authenticated"

# GitLab
ssh -T git@gitlab.com
```

---

## First steps

```bash
# Create new repository
mkdir my-project && cd my-project
git init

# Clone existing one
git clone <URL>
git clone <URL> my-folder          # into named folder
git clone --depth 1 <URL>          # only latest commit

# Check status
git status

# Add files and commit
git add .
git commit -m "Initial commit"

# Connect remote repository
git remote add origin git@github.com:ponfertato/my-project.git
git push -u origin main
```

---

## Useful aliases (optional)

```bash
# Add to ~/.gitconfig or run commands:
git config --global alias.co checkout
git config --global alias.br branch
git config --global alias.ci commit
git config --global alias.st "status -s"
git config --global alias.lg "log --oneline --graph --all"

# Usage:
git st      # instead of git status -s
git lg      # pretty log
git co -b feature  # create and switch to branch
```

---

## Troubleshooting

```bash
# Git not found after install
→ Restart terminal / computer

# "Permission denied (publickey)" error
→ Ensure public key is added to GitHub/GitLab profile
→ Test: ssh -T git@github.com

# Line ending conflicts (CRLF/LF)
→ Set core.autocrlf for your OS (see above)

# Reset all config to defaults
git config --global --unset-all user.name
git config --global --unset-all user.email
# Or delete: ~/.gitconfig
```

---

## Links

- 🌐 [Official site](https://git-scm.com)
- 📘 [Pro Git book (free)](https://git-scm.com/book/en/v2)
- 🎮 [Interactive tutorial](https://learngitbranching.js.org)
- 🔑 [.gitignore generator](https://gitignore.io)
