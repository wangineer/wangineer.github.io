---
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



## How to Git Git (Install/Download)

Download Git [**here**][gitdownload]

<hr>


> Macbooks have `git` installed automatically
{: .prompt-info }


### Linux
```shell
apt-get install git   (version will not be the same as what is shared on website likely)
```


### Windows

Go to [**https://git-scm.com**][gitdownload] and click to download.  The site will know you are running windows.

> Once installed you can open the program `git bash`  This will allow you to use all of the linux commands usedto work with Git.
{: .prompt-tip }


### Verify Git Version: 
Once installed you will want to verify the version you are running

```bash
test@servername:~/git/github$ git --version
git version 2.34.1
```


## Now what you've Gotten Git, you can finally Git Started (Video 3)

### How to Create a Local Git Repository

#### Create a Folder and then Init
```shell
mkdir localrepo
cd localrepo
git init
```

Below will be outputted
```shell
Initialized emtpy Git respository in /Users/test/Desktop/localrepo/.git/
localrepo git:(master)    <-- You see (master) if you are on windows bash
```

>If you do `ls -la` you will see a `.git` folder.  This is where all your saves are located
{: .prompt-tip }
```
test@servername:~$ ls -la | grep git
drwxrwxr-x  8 test test 4096 Oct 25 23:46 .git
```

## Understanding Git files

### .gitconfig
This is where your system wide Git configurations are stored.  One of the first things that needs to be done after installing `git` is to apply your name and email.  If you are going to commit anything this needs to be set. 

#### Example file
``` 
test@servername:~$ cat .gitconfig
[user]
        name = TESTNAME
        email = testname@email.com
test@servername:~$ git config --global --list
user.name=TESTNAME
user.email=testname@email.com
```


### Other files and folders

Here is a list of the files/folders and a description
- config - basic config how to use - you can use 
- decription - description of repo
- Head - Current snapshot  you are on 
- Hooks - Something you can run automatically
- INFO - exclude file - Patterns of files you want to keep out of the Repo like credict card files SSH files
- objects - packs like make it smaller
- refs - check out instances in time


## Your first commands to start Git'ting
You will need to set a user name and email.  
```bash
git config --global user.name 'Test Name'
git config --global user.email@mail.com
```



## How to Remove Git being managed from a folder (delete the .git directory)
 - Not sure how but i have .git in upper folder and current folder
 ----------------------REVIEW--------------------------------------------------------
```
rm -rm .git
```


## How to Clone repo from Github
Go to the website and click the "Clone or Download" He did this in his regular desktop folder  it created a folder called "fetch" and it becomes a repo automcatically with the folder added.  YOU DO NOT NEED TO Configure "git init"
```
$git clone https://github.com/github/fetch.git
$cd fetch
```
output will show (In windows)
```
fetch git:(master)
```

You will also see new files generated from this compared to just having locally
- packed refs - compressed
- logs - thats whats happening over time of the snapshots
- index - makes sure you can do a lookup of whats happened over time very quickly



## What have we learned
## How to Git up and Running quick
1. Install Git

2. Configure the .gitconfig file
- git config --global user.name 'Test Name'
- git config --global user.email test@mail.com

3. a. Create Git Locally
Create a Folder and then Init
```
mkdir localrepo
cd localrepo
git init
```

3. b. How to download from Github (You do not need to do 'git init')
Example of downloading a repo from Github Fetch
$git clone https://github.com/github/fetch.git
$cd fetch



Either clone Repo. or start new setup and then push to github







## Quick Commands
Lists all files including hidden in the directory
```
ls -la
```

How to set the default editor to vim
```
git config --global core.editor vim
```

If you want notepad++ you need full path like in windows
  - Note the '\ '  This is an escape regex character to symbolize a space
```
git config --global core.editor C://Program Files\ Files/Notepad++.exe
```

If you want to list the git config command and get a few more deatis. 
```
git config --list
```


[gitdownload]: https://git-scm.com