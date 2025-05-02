---
title: Gittin on Git
description: Git on with it
date: 2025-05-01 12:00:00:01 +/-0000
categories: [Git, CI/CD]
tags: [git,init]     # TAG names should always be lowercase
toc: true
comments: active
image:
  path: '/assets/img/headers/gitinit.webp'
pin: false
published: true
---
## Scenario
I'm a Network Engineer that wants to learn git.  Below are some notes I can go to so i don't forget.  Hopefully this will help other people trying to get into it.    

## How to Git Git (Install/Download)

Website for Git (git-scm.com  & git-scm.com/download/linux)

Mac: has it installed already
Linux: sudo apt-get install git   (version will not be the same likely)
                 wget
Windows: git-scm.com click on the download
   Open program "git bash" and that gives you all the linux commands


Verify Version: 
```
test@servername:~/git/github$ git --version
git version 2.34.1
```

## .gitconfig
This is where you system wide configurations store your "name" and "email" that is used for config change.
etc/gitconfig  (I couldn't find anything at this location)
/home/test   (this is location 'test' is the username logged in)

```
test@servername:~$ cat .gitconfig
[user]
        name = TESTNAME
        email = testname@email.com
test@servername:~$ git config --global --list
user.name=TESTNAME
user.email=testname@email.com
```

How to set a Global Username and email
```
git config --global user.name 'Test Name'
git config --global user.email@mail.com
```

How to verify confiuration is set correctly

## Now what you've Gotten Git, you can finally Git Started (Video 3)

### How to Create a Local Git Repository

Create a Folder and then Init
```
mkdir localrepo
cd localrepo
git init
```
Below will be outputted
Initialized emtpy Git respository in /Users/test/Desktop/localrepo/.git/
localrepo git:(master)    <-- You see (master) if you are on windows bash

If you do "ls -la" you will see a ".git" folder.  This is where all your saves are located
```
test@servername:~$ ls -la | grep git
drwxrwxr-x  8 test test 4096 Oct 25 23:46 .git
```

If you hop out of that directory, there is no .git folder

- config - basic config how to use - you can use 
- decription - description of repo
- Head - Current snapshot  you are on 
- Hooks - Something you can run automatically
- INFO - exclude file - Patterns of files you want to keep out of the Repo like credict card files SSH files
- objects - packs like make it smaller
- refs - check out instances in time


How to Remove Git being managed froma folder (delete the .git directory)
 - Not sure how but i have .git in upper folder and current folder
 ----------------------REVIEW--------------------------------------------------------
```
rm -rm .git
```


### How to Clone repo from Github
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



#Wrap Up -------------------------------------------------------
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