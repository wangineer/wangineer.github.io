---
title: Windows Update Puts Hyper-V in Saved State
description: Nothing a little Powershell can't fix.
date: 2024-08-05 12:00:00:01 +/-0000
categories: [Virtualization, Hyper-V]
tags: [powershell,vms,homelab]     # TAG names should always be lowercase
toc: true
comments: active
ping: true
image:
  path: '/assets/img/headers/WindowsUpdatesavedstate.webp'
pin: false
# published: false
---

## The Problem - VM's go into Saved State after Windows Update

Prior to having my homelab server, I used my headless PC to host all my virtual machines. Things were running fine for years, until one day my PC decided to complete a routing update.  Except during this update, I found all my VM's stuck in a "saved" state.  I tried to "Start" the VM's, restart services, and even reboot my pc.  Nothing seemed to work.  

## Troubleshooting Efforts
![Desktop View](/assets/img/savedstateerror.webp)

Every time I tried to start a VM, I recieved the error "The application encourntered an error while attempting to change the state of Windows 10 Tech Preview"
 - Windows 10 Tech Preview failed to restore virtual machine state.

Disregard the mention of "Tech Preview" as that may have been the name of the VM, it was trying to bringup. 

## The Band-aid
After Googling, I found that if ran the below powershell script and rebooted my physical computer, I could then Start all my VM's. 
```Powershell
bcdedit /set hypervisorlaunchtype auto
```

## The Fix
I finally moved all of the virtual machines to my Homelab Server
