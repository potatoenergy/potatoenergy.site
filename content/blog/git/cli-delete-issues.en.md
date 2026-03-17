---
author: ["Potato Energy Team", "ponfertato"]
categories: ["git", "github", "automation", "guide"]
date: "2026-03-17T09:30:00+03:00"
description: "Bulk delete old GitHub Issues by labels and date using GitHub CLI and jq."
draft: false
series: ["Git Tips"]
slug: "github-cli-delete-issues-en"
tags: ["git", "github", "cli", "jq", "automation", "cleanup"]
title: "GitHub CLI: Bulk Delete Old Issues"
---

Long-running projects accumulate outdated issues: bugs for old versions, features that are no longer relevant, test tickets. Manual deletion is slow and tedious. This script automatically finds and deletes (or marks) old issues by specified labels.

> 💡 Script uses **dry-run by default** — shows what would be deleted, without actual deletion.

---

## 📦 Script: delete-issues.sh

### Full code

```bash
#!/bin/bash
# Delete old GitHub Issues by labels and date
# Usage: ./delete-issues.sh [--execute]

set -euo pipefail

# === CONFIG ===
REPO="owner/repo"                              # Repository in owner/repo format
LABELS='label1,label2,label3'                  # Labels to filter (comma-separated)
CUTOFF="2025-12-31T23:59:59Z"                 # Delete issues created BEFORE this date
DRY_RUN=true                                   # true = preview only, false = actually delete

# Parse arguments
if [[ "${1:-}" == "--execute" ]]; then
    DRY_RUN=false
    echo "⚠️  Mode: ACTUAL DELETION"
else
    echo "ℹ️  Mode: DRY RUN (nothing will be deleted)"
fi

echo "🔍 Searching issues in $REPO with labels: $LABELS, created before $CUTOFF"
echo "---"

# Fetch and filter issues
gh issue list --repo "$REPO" --state all --limit 1000 \
  --json number,title,createdAt,labels,url | \
  jq -r --arg labels "$LABELS" --arg cutoff "$CUTOFF" '
    ($labels | split(",")) as $label_array |
    .[] |
    select(.createdAt < $cutoff) |
    select(.labels | map(.name) | any(. as $l | $label_array | index($l))) |
    "\(.number)|\(.title)|\(.url)"
  ' | \
  while IFS='|' read -r number title url; do
    if [[ "$DRY_RUN" == "true" ]]; then
      echo "[DRY RUN] ##$number — $title"
      echo "          $url"
    else
      echo "🗑  Deleting ##$number — $title"
      gh issue delete "$REPO" "$number" --yes
      sleep 1  # Small pause to avoid rate limits
    fi
  done

echo "---"
echo "✅ Done"
```

### Install dependencies

```bash
# GitHub CLI
# Windows (winget):
winget install GitHub.cli

# Linux (Ubuntu/Debian):
sudo apt install gh

# macOS (Homebrew):
brew install gh

# Authenticate
gh auth login

# jq (JSON processor)
# Windows (winget):
winget install jq.jq

# Linux:
sudo apt install jq

# macOS:
brew install jq
```

### Run

```bash
# Make script executable
chmod +x delete-issues.sh

# DRY RUN (safe mode — preview only)
./delete-issues.sh

# ACTUAL DELETION (add --execute flag)
./delete-issues.sh --execute
```

---

## 🔍 How it works

### Step-by-step breakdown

| Step | Command                    | What it does                                                         |
| ---- | -------------------------- | -------------------------------------------------------------------- |
| 1    | `gh issue list --json ...` | Fetches issues as JSON with fields: number, title, date, labels, URL |
| 2    | `jq -r ...`                | Filters: date < cutoff AND has at least one of specified labels      |
| 3    | `while read ...`           | Processes each matching issue                                        |
| 4    | `gh issue delete`          | Deletes issue (only if `DRY_RUN=false`)                              |

### jq filter logic

```jq
($labels | split(",")) as $label_array |  # Split label string into array
.[] |                                      # For each issue
select(.createdAt < $cutoff) |            # Only if created before cutoff
select(
  .labels | map(.name) |                  # Get list of label names
  any(. as $l | $label_array | index($l)) # Check if any matches our labels
) |
"\(.number)|\(.title)|\(.url)"            # Format output
```

**Why this way**:

- `split(",")` — allows specifying labels as single string
- `any(...)` — checks for **any** of specified labels (logical OR)
- Pipe-delimited output — reliable way to pass titles with spaces

---

## ⚙️ Adapt for your use case

### Example 1: Delete old bugs

```bash
REPO="myorg/myproject"
LABELS='bug,critical'
CUTOFF="2024-01-01T00:00:00Z"  # Delete bugs before 2024
```

### Example 2: Close instead of delete

Replace deletion block with close:

```bash
# Instead of gh issue delete:
echo "🔒 Closing ##$number — $title"
gh issue edit "$REPO" "$number" --state closed
```

### Example 3: Export before delete (backup)

```bash
# Add before deletion:
echo "💾 Exporting ##$number"
gh issue view "$REPO" "$number" --json title,body,comments >> backup-issues.jsonl
```

---

## 🛡 Safety

### Mandatory checks before running

```bash
# 1. Verify selected issues are actually the ones you want
./delete-issues.sh | grep -c "\[DRY RUN\]"   # Count matches

# 2. Ensure no critical issues in list
./delete-issues.sh | grep -i "critical\|important"

# 3. Check permissions
gh api /repos/$REPO | jq '.permissions.admin'  # Should be true
```

### GitHub API rate limits

| Type            | Limit              | How to stay under           |
| --------------- | ------------------ | --------------------------- |
| Authenticated   | 5000 requests/hour | `sleep 1` between deletions |
| Unauthenticated | 60 requests/hour   | Always use `gh auth login`  |

**Check remaining quota**:

```bash
gh api rate_limit | jq '.resources.core'
```

---

## 📊 Alternatives and extensions

### Preview only (no deletion)

```bash
gh issue list --repo "$REPO" --state all --limit 1000 \
  --json createdAt,labels | \
  jq -r --arg labels "$LABELS" --arg cutoff "$CUTOFF" '
    ($labels|split(",")) as $label_array |
    [.[] | select(.createdAt < $cutoff) |
     select(.labels | map(.name) | any(. as $l | $label_array | index($l)))] |
    length
  '
```

### Delete by author + labels

```jq
select(.author.login == "bot-user") |
select(.labels | map(.name) | any(. as $l | $label_array | index($l)))
```

### Export to CSV before delete

```bash
# Add to jq output:
"\(.number),\"\(.title)\",\(.createdAt),\(.url)"
```

---

## ⚠️ Common issues

```bash
# "gh: command not found"
→ Install GitHub CLI: https://cli.github.com

# "jq: error: syntax error"
→ Check quotes: use single quotes for jq, double for bash

# "HTTP 404: Not Found"
→ Verify repo name: must be owner/repo
→ Ensure token has issues:read permission

# "rate limit exceeded"
→ Wait for reset or add delay: sleep 2

# Deleted wrong issues!
→ Always run without --execute first
→ Backup first: gh issue list --json ... > backup.json
```

---

## 🗂 Pre-run checklist

- [ ] Installed `gh` and `jq`, ran `gh auth login`
- [ ] Verified repo permissions: `gh api /repos/owner/repo`
- [ ] Tested in dry-run mode, reviewed issue list
- [ ] Confirmed no critical issues in deletion list
- [ ] Exported backup if needed: `gh issue list --json ... > backup.json`
- [ ] Only then — run with `--execute`

---

## Links

- 🐙 [GitHub CLI Docs](https://cli.github.com/manual/)
- 🔍 [gh issue list](https://cli.github.com/manual/gh_issue_list)
- 🗑️ [gh issue delete](https://cli.github.com/manual/gh_issue_delete)
- 🧰 [jq Manual](https://jqlang.github.io/jq/manual/)
- 🔐 [GitHub API Rate Limits](https://docs.github.com/en/rest/rate-limit)
