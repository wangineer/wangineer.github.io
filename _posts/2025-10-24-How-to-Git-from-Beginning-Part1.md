---
title: "Gittin’ on Git (Git Series: Part 1)"
description: "A Network Engineer begins a beginner-friendly journey into Git—learning how version control can streamline documentation, collaboration, and troubleshooting in an evolving network landscape."
date: 2025-10-24 12:00:00 +/-0000
categories: [Series, Git]
tags: [git, gitinit, github, network-engineering, git-basics]
toc: true
comments: true
image:
  path: '/assets/img/headers/gitaddremove.webp'
  alt: 'Terminal showing git add, git rm, git commit - commands used when using git'
#  lqip: true
pin: false
published: true
permalink: /series/git/gittin-on-git/
seo:
  title: "Learning Git as a Network Engineer | Git Series Part 1"
  description: "A hands-on beginner’s guide for network engineers to install, configure, and understand Git—laying the foundation for better version control and documentation."
  canonical: "https://wangineer.com/series/git/gittin-on-git/"
  keywords: ["git basics", "git tutorial", "git setup", "network engineer git", "version control guide"]
---

## Why Git Matters (Scenario)

As network engineers, we work in a field that never stops evolving. Staying ahead often means adopting tools that help us work smarter—not harder. While Git isn’t a core requirement in my current role, its power for tracking changes, collaborating effectively, and documenting progress has caught my attention.

This blog series is my beginner’s journey into Git. My goal? To explore how version control fits into a network engineer’s workflow and to create a reference I (and others) can come back to when troubleshooting or experimenting.  

If you’ve ever thought, *“I should probably learn Git someday,”* this series is your sign to start now. Let’s learn it together.

---

## How to Install Git

Before diving in, make sure you have a terminal (local machine or homelab) and an internet connection to download Git.  


If you plan to sync your work online, a GitHub or GitLab account is optional but helpful. In future posts I will be using GitHub.

The first step is to get Git.  You can do this by going to the website. 

Download Git from [**git-scm.com**](https://git-scm.com){:target="_blank"} — the official Git website.

### 🖥️ macOS
```bash
# Git is usually preinstalled on macOS
git --version
```

> Note: If Git is not installed, you can download it from the Git website.
{: .prompt-info }

### 🐧 Linux
```bash
sudo apt-get install git
```

> Note: The version you get may differ from what is listed on the Git website.
{: .prompt-info }

### 🪟 Windows
Go to [**https://git-scm.com**](https://git-scm.com){:target="_blank"} and click to download. The site will detect your Windows version.

> Once installed, open **Git Bash**. This allows you to run Linux-style Git commands.
{: .prompt-tip }

---

### Verify Git Version
Once you have Git installed, you can open a the command prompt or Git Shell and issue the below command.
```bash
git --version
```

Example output:
```text
git version 2.34.1
```

---

## How to Get Started
To get started on managing version control, we need to first have some files to track changes with.  We can do this by creating root folder for your project.  Below i'm calling it "localrepo"

### Create a Folder and Initialize Git
```bash
mkdir localrepo
cd localrepo
git init
```

**Example Output:** This is what is shown when you complete the first `git init`
```text
Initialized empty Git repository in /Users/test/Desktop/localrepo/.git/
```

> You may see `(master)` or `(main)` depending on your Git version.
{: .prompt-info }

### Assign User and Email
Now that we have Git initiated, you need to assign a name and email.  This is used for every time you commit a change, these artifacts get associated with your change.  If you don't set this, then you will get an error message when you complete your first commit stating you haven't set your Name and Email.
```bash
git config --global user.name "Test Name"
git config --global user.email "test@mail.com"
```

---

## Understanding Git Files
When you complete the first "git init" a new hidden folder gets created.  to see the hidden files you can issue the  `ls -la` command.

### Hidden Git Directory
```bash
ls -la | grep .git
drwxrwxr-x  8 test test 4096 Oct 25 23:46 .git
```

> This folder contains all Git data for your repository.
{: .prompt-tip }

### .gitconfig
One of the files `.gitconfig` show your name and email you set
```text
[user]
    name = TESTNAME
    email = testname@email.com
```

List all global configurations:
```bash
git config --global --list
```

### Other Important Folders in `.git`

| File/Folder     | Description |
|-----------------|-------------|
| `config`        | Repository-specific configuration file |
| `description`   | Repository description |
| `HEAD`          | Points to current branch reference |
| `hooks/`        | Scripts that run automatically on Git events |
| `info/`         | Contains `exclude` file for ignored patterns |
| `objects/`      | Stores commits, files, and history |
| `refs/`         | References to commits and branches |

---

## Quick Recap

1. **Install Git**  
2. **Initialize a Repository**
3. **Set User and Email**

---

## Quick Commands Reference
```bash
# List all files including hidden
ls -la

# Install Git on Linux
sudo apt-get install git

# Check Git version
git --version

# Initialize a repository
mkdir localrepo
cd localrepo
git init

# Set user name and email
git config --global user.name "Test Name"
git config --global user.email "test@mail.com"

# View Git configuration
git config --list
cat ~/.gitconfig

# Set default editor
git config --global core.editor vim

# Set Notepad++ on Windows
git config --global core.editor "C:\\Program Files\\Notepad++\\Notepad++.exe"
```

---

## Call to Action

Try this now in a folder on your machine or homelab. Drop a comment below with your OS and any error messages. Myself or someone may be able to assist you. Stay tuned for Part 2, where we’ll cover adding, removing, and committing files.

---

## Resources

- [Official Git Website](https://git-scm.com){:target="_blank"}
- [Pro Git Book (Free)](https://git-scm.com/book/en/v2){:target="_blank"}

## 🚀 Git Series Navigation

Stay up to date with the full series and follow along as we dive deeper into Git:

| Part | Title | Link |
|------|-------|------|
| 1 | Gittin’ on Git | [Part 1](https://wangineer.com/series/git/gitten-on-git/) |
| 2 | Lets Git Started (TBD) | [Part 2](#) |
| 3 | Behind Git (TBD) | [Part 3](#) |
| 4 | Git into Branching (TBD)  | [Part 4](#) |
| 5 | Three-way merge and Rebasing (TBD)  | [Part 5](#) |
| * | More to Come (TBD) | [Part #](#) |


> Tip: Bookmark this series so you can quickly reference each part as you learn.
{: .prompt-tip }