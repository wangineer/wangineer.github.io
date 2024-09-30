---
title: How to Replace Mirror'd Hard Drive in ZFS Proxmox
description: Modulating a successful pass!
date: 2024-08-15 12:00:00:01 +/-0000
categories: [Proxmox, Troubleshoot]
tags: [server,raid,mirror,zfs,zpool]     # TAG names should always be lowercase
toc: true
comments: active
ping: true
image:
  path: '/assets/img/headers/hello.webp'
pin: false
# published: false
---
Stats:
 - ZFS Mirror using two 500gb hard drives
 - Proxmox Version 7.4-16

Scenario: I recieved multiple emails over many months stating my Proxmox Mirror was Degraded and couldn't correct errors.  


Pre-steps:  PLace the pre steps to clear the counters. 


I powered off my server and pulled out the faulty hard drive.
Note: One thing I do when I install my drives, is that I took down the serial number, and put it into an excel sheet noting what slot I placed it in. 


I ordered the correct Hard drive off of amazon. 

```
root@pve1:~# zpool status
  pool: zfs500mirror
 state: DEGRADED
status: One or more devices are faulted in response to persistent errors.
        Sufficient replicas exist for the pool to continue functioning in a
        degraded state.
action: Replace the faulted device, or use 'zpool clear' to mark the device
        repaired.
  scan: scrub repaired 104K in 01:06:13 with 0 errors on Sun Sep  8 01:30:14 2024
config:

        NAME                          STATE     READ WRITE CKSUM
        zfs500mirror                  DEGRADED     0     0     0
          mirror-0                    DEGRADED     0     0     0
            ata-ST3500312CS_9VV0PEMS  FAULTED      6   630    12  too many errors
            ata-ST3500312CS_9VV0PREA  ONLINE       0     0     0

errors: No known data errors
```






I watched this video