---
title: "Git Basics for Network Engineers — add, commit, rm, mv, amend (Git Series: Part 2)"
description: "Practical, hands-on examples of git add, git rm, git mv, git commit, and git commit --amend — explained for network engineers."
date: 2025-10-25 12:00:00 +/-0000
categories: [Series, Git]
tags: [git-basics, git-add, git-commit, git-rm, git-amend, git-mv, git-for-network-engineers, version-control, devops, homelab]
toc: true
comments: true
pin: false
published: true
image:
  path: '/assets/img/headers/gitpart2.webp'
  alt: 'Shell window with Git commands being shown'
permalink: /series/git/git-basics/
seo:
  title: "Git Basics for Network Engineers — Add, Commit, Remove, Amend | Git Series Part 2"
  description: "Learn how network engineers can use Git commands like git add, git commit, git rm, git mv, and git commit --amend to manage configuration and documentation efficiently."
  canonical: "https://wangineer.com/series/git/git-basics/"
  keywords: ["git add command", "git commit guide", "git rm tutorial", "git amend example", "network engineer git", "git workflow basics", "learn git 2025", "version control tutorial", "git for beginners", "git cli examples"]
---

| Part | Title | Link |
|------|-------|------|
| 1 | Gittin’ on Git | [Part 1](https://wangineer.com/series/git/gittin-on-git/) |
| 2 | Git Basics | [Part 2](https://wangineer.com/series/git/git-basics/) |
| 3 | Behind Git (TBD) | [Part 3](#) |
| 4 | Git into Branching (TBD) | [Part 4](#) |
| 5 | Three-way merge and Rebasing (TBD) | [Part 5](#) |
| * | More to Come (TBD) | [Part #](#) |

---

## Let’s Git Started (Scenario)

In [Part 1](https://wangineer.com/series/git/gittin-on-git/), we installed and configured Git.  
Now it’s time to **get hands-on**.

As network engineers, we live in real-time — editing running configs, capturing snapshots, and trying not to break production. Git gives that same level of control for your **scripts, templates, and documentation**. Every change is logged, reversible, and traceable.

Let’s walk through how Git tracks files, what it means for them to be *untracked* or *staged*, and how everyday commands like `git add`, `git rm`, `git commit`, `git mv`, and `git commit --amend` fit into your workflow.

---

## States of Git

> Use `git status` anytime to see what’s happening inside your repo.
{: .prompt-info }

### Untracked vs Tracked Files
- **Untracked** — Files Git isn’t monitoring yet.  
- **Tracked** — Files Git already knows about and monitors for changes.

### File States for Tracked Files
1. **Unmodified** — Same as the last commit.  
2. **Modified** — Changed but not yet staged.  
3. **Staged** — Added to the *staging area* with `git add`, waiting for commit.

---

## Checking Git Status

Use this command frequently — it’s your status dashboard.

```bash
git status
```

Output typically includes:
- Current branch  
- Modified but unstaged files  
- Staged files ready to commit  
- Untracked files  
- Whether your branch is ahead/behind remote  
- Merge conflict details (if any)

Short format option:

```bash
git status -s
```

> `M` = Modified, `A` = Added, `??` = Untracked  
{: .prompt-tip }

---

## Git Add

Stage files to include them in your next snapshot. This normally needs to be completed when you created a new file or folder

```bash
git add readme.md
```

Add everything in the folder:
```bash
git add *
```

Check your changes:
```bash
git status
```

---

## Git Commit

Once your files are staged, save the snapshot.

```bash
git commit -m "Add initial README and diagrams"
```

Every commit should have a clear, short message — think *“why”* more than *“what.”*

---

### Git Commit — Amend

Forgot something in the last commit? Instead of making a new one, amend it:
For example if you created file1.txt and file2.txt and only completed a ***git add file1.txt*** before committing, it would have left out file2.txt from being tracked.  Instead of completing a whole new commit adding that single file, you can ammend the commit. When you issue the below command it opens the last commit message file and you can ammend the text to include the file. 

```bash
git add file2.txt
git commit --amend
```

> This reopens your previous commit message in the default text editor.
{: .prompt-info }

Set your preferred editor:
```bash
git config --global core.editor vim
```

Windows example:
```bash
git config --global core.editor C://Program Files\ Files/Notepad++.exe
```
> If you want notepad++ you need full path like in windows
  - Note the '\ '  This is an escape regex character to symbolize a space
{: .prompt-info }


---

## Git Move (mv)

To move a file and keep Git history intact:

```bash
git mv filename.txt subdir/
```

> The file’s rename or move will show in the next commit.
{: .prompt-tip }

---

## Git Remove

Remove files from tracking — or delete them entirely.

### Remove from tracking only
(Keep the file on disk.)
```bash
git rm --cached filename.txt
git rm --cached -r foldername/
```

### Force remove tracked file
(Delete it from both repo and disk.)
```bash
git rm -f oldfile.txt
```

### Remove Git tracking completely
(Remove `.git` folder — use caution.) This removes tracking for the entire project.
```bash
rm -rf .git
```

---

## Restore a File Before Committing
Accidentally edited the wrong file? Revert it before staging:
This allows you to restore back to its last committed states.

```bash
git restore filename.txt
```

---

## Log Quick View

If you want to take a look at the commit history you can use the below command.  We will discuss this in future posts in more:

```bash
git log --oneline
```

> Great for quick overviews or troubleshooting what changed.
{: .prompt-info }

---

## Quick Reference Options

### **Option B — Table View**

| Command | Description |
|----------|--------------|
| `git add readme.md` | Add specific file to staging |
| `git add *` | Add all files |
| `git commit -m "message"` | Create commit with message |
| `git commit --amend` | Modify last commit |
| `git mv file.txt subdir/` | Move or rename a file |
| `git rm --cached file.txt` | Untrack file but keep it locally |
| `git rm -f file.txt` | Force remove from repo and disk |
| `git restore file.txt` | Undo local changes |
| `git log --oneline` | Show commit history (short) |
| `git status -s` | Show status summary |

---

## Recap

You’ve learned how to:
1. Track and manage files with `git add`, `git commit`, and `git rm`  
2. Modify existing commits with `--amend`  
3. Restore mistakes before committing  
4. Move and clean up files safely  
5. Read quick logs for validation  

These form the **core muscle memory** for working confidently with Git.

In **Part 3**, we’ll peek *behind the curtain* to explore what’s inside the `.git` directory and how snapshots, hashes, and object storage really work.

---

| Part | Title | Link |
|------|-------|------|
| 1 | Gittin’ on Git | [Part 1](https://wangineer.com/series/git/gittin-on-git/) |
| 2 | Git Basics | [Part 2](https://wangineer.com/series/git/git-basics/) |
| 3 | Behind Git (TBD) | [Part 3](#) |
| 4 | Git into Branching (TBD) | [Part 4](#) |
| 5 | Three-way merge and Rebasing (TBD) | [Part 5](#) |
| * | More to Come (TBD) | [Part #](#) |