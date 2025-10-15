---
layout: post
title: Gittin on Git
description: Discover how a Network Engineer embarks on a beginner-friendly journey to learn Git—exploring its potential to adapt to a changing tech landscape, document key lessons, and troubleshoot effectively in this first post of an insightful series.
date: 2025-05-01 12:00:00:01 +/-0000
categories: [series, git]
tags: [gitinit,github]     # TAG names should always be lowercase
toc: true
comments: active
image:
  path: '/assets/img/headers/gitinit.webp'
pin: false
published: true
---
## Scenario
As network engineers, we know our industry is constantly evolving, and staying ahead means embracing tools that can enhance our workflows. While Git isn’t a requirement in my current role, its potential to streamline version control, collaboration, and problem-solving has piqued my interest. This blog series is my way of diving into the world of Git from a beginner's perspective—documenting my journey, sharing insights, and creating a resource to reference when things (inevitably) go wrong. Whether you’re curious about Git or considering adding it to your skillset, I invite you to join me on this learning adventure.


## How to get Git

Download Git [**here**](https://git-scm.com){:target="_blank"} (https://git-scm.com)

### Macbook
> Macbooks have `git` installed automatically
{: .prompt-info }


### Linux
```shell
apt-get install git   (version will not be the same as what is shared on website likely)
```

### Windows

Go to [**https://git-scm.com**][gitdownload] and click to download.  The site will know you are running windows.

> Once installed you can open the program `git bash`  This will allow you to use all of the linux commands used to work with Git.
{: .prompt-tip }


### Verify Git Version: 
Once installed you will want to verify the version you are running

```bash
test@servername:~/git/github$ git --version
git version 2.34.1
```

## How to get started

### Create a Folder and then Init
```shell
mkdir localrepo
cd localrepo
git init
```

### Below will be outputted
```shell
Initialized emtpy Git respository in /Users/test/Desktop/localrepo/.git/
localrepo git:(master)    <-- You see (master) if you are on windows bash
```
### How to assign User and Email account to start commit'ing
You will need to set a user name and email.  
```bash
git config --global user.name 'Test Name'
git config --global user.email@mail.com
```

## Understanding Git files

### Hidden Git Directory 
When you complete the first "git init" a new folder gets created

>If you do `ls -la` you will see a `.git` folder.  This is where all your saves are located
{: .prompt-tip }
```
test@servername:~$ ls -la | grep git
drwxrwxr-x  8 test test 4096 Oct 25 23:46 .git
```

### .gitconfig
This is where your system wide Git configurations are stored.  One of the first things that needs to be done after installing `git` is to apply your name and email.  If you are going to commit anything this needs to be set. This was completed in the previous section. 

### Example file and command to see all config
``` 
test@servername:~$ cat .gitconfig
[user]
        name = TESTNAME
        email = testname@email.com

test@servername:~$ git config --global --list
user.name=TESTNAME
user.email=testname@email.com
```

## Understanding Other files and folders

Here is a list of the files/folders and a description
- config - basic config how to use - you can use 
- decription - description of repo
- Head - Current snapshot  you are on 
- Hooks - Something you can run automatically
- INFO - exclude file - Patterns of files you want to keep out of the Repo like credict card files SSH files
- objects - packs like make it smaller
- refs - check out instances in time


### Recap: How to Git up and Running quick
1. Install Git

2. a. Create Git Locally
Create a Folder and then Init
```
mkdir localrepo
cd localrepo
git init
```

3. Configure the .gitconfig file
```
- git config --global user.name 'Test Name'
- git config --global user.email test@mail.com
```

## Quick Commands
###Lists all files including hidden in the directory
```
ls -la
```

### How to set the default editor to vim
```
git config --global core.editor vim
```

### If you want notepad++ you need full path like in windows
  - Note the '\ '  This is an escape regex character to symbolize a space
```
git config --global core.editor C://Program Files\ Files/Notepad++.exe
```

### If you want to list the git config command and get a few more deatis. 
```
git config --list
```

## Resources
[**https://git-scm.com**](https://git-scm.com){:target="_blank"}