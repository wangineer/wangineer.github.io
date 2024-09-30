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
  path: '/assets/img/headers/proxmox.webp'
pin: false
# published: false
---
Stats:
 - ZFS Mirror using two 500gb hard drives
 - Proxmox Version 7.4-16

Scenario: I recieved multiple emails over many months stating my Proxmox Mirror was Degraded and couldn't correct errors.  


Pre-steps:  Place the pre steps to clear the counters. 


I powered off my server and pulled out the faulty hard drive.
Note: One thing I do when I install my drives, is that I took down the serial number, and put it into an excel sheet noting what slot I placed it in. 


I ordered the correct Hard drive off of amazon. 


Below command "zpool status"  shows the status of the pool and we see that hard drive with serial number 9VV0PEMS is in a Faulty State
  You can clear counters.  I think its in the video on how to do it to see if its acutally an issue

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




To Replace the hard drive it think i did it the wrong way, but I have the output on how to fix it if you don't do it correct. 

First you need to find the faulty Drive
It may be a good idea to get the below output

```
ls -ahlp /dev/disk/by-id/ 
```



I powered off the server and pulled the hard drive out.  then powered it back on. 

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
            15090045841374906569      FAULTED      6   630    12  was dev/disk/by-id/ata-ST3500312CS_9VV0PEMS-part1
            ata-ST3500312CS_9VV0PREA  ONLINE       0     0     0

errors: No known data errors
```


You will need the "was" section


To get rid of the old reference and add the new hard drive I completed the below command, because i saw the drive refereced "sdb"

zpool replace "Pool Name" /dev/disk/by-id/"name you copied above was" "New location of harddrive

```
zpool replace zfs500mirror /dev/disk/by-id/ata-ST3500312CS_9VV0PEMS-part1 /dev/sdb
```


Once i completed i did zpool status and saw it was replacing and resilvering
```
root@pve1:~# zpool status
  pool: zfs500mirror
 state: DEGRADED
status: One or more devices is currently being resilvered.  The pool will
        continue to function, possibly in a degraded state.
action: Wait for the resilver to complete.
  scan: resilver in progress since Mon Sep 30 13:21:06 2024
        82.4G scanned at 54.3M/s, 29.4G issued at 19.3M/s, 82.4G total
        29.9G resilvered, 35.61% done, 00:46:49 to go
config:

        NAME                          STATE     READ WRITE CKSUM
        zfs500mirror                  DEGRADED     0     0     0
          mirror-0                    DEGRADED     0     0     0
            replacing-0               DEGRADED     0     0     0
              15090045841374906569    UNAVAIL      0     0     0  was /dev/disk/by-id/ata-ST3500312CS_9VV0PEMS-part1
              sdb                     ONLINE       0     0     0  (resilvering)
            ata-ST3500312CS_9VV0PREA  ONLINE       0     0     0

errors: No known data errors
```


You can see the above name doesn't match the "ata" name.  This can cause issues during a reboot i suspect.  you always want to use the id. 

You can now see below completed and all is good except the name
```
root@pve1:~# zpool status
  pool: zfs500mirror
 state: ONLINE
status: Some supported and requested features are not enabled on the pool.
        The pool can still be used, but some features are unavailable.
action: Enable all features using 'zpool upgrade'. Once this is done,
        the pool may no longer be accessible by software that does not support
        the features. See zpool-features(7) for details.
  scan: resilvered 84.0G in 01:11:51 with 0 errors on Mon Sep 30 14:32:57 2024
config:

        NAME                          STATE     READ WRITE CKSUM
        zfs500mirror                  ONLINE       0     0     0
          mirror-0                    ONLINE       0     0     0
            sdb                       ONLINE       0     0     0
            ata-ST3500312CS_9VV0PREA  ONLINE       0     0     0

errors: No known data errors
```




If you want to replace this you need to complete a detach and then attach. 

```
root@pve1:~# zpool detach zfs500mirror sdb
```

You can now see the "sdb" removed from the mirror
```
root@pve1:~# zpool status
  pool: zfs500mirror
 state: ONLINE
status: Some supported and requested features are not enabled on the pool.
        The pool can still be used, but some features are unavailable.
action: Enable all features using 'zpool upgrade'. Once this is done,
        the pool may no longer be accessible by software that does not support
        the features. See zpool-features(7) for details.
  scan: resilvered 84.0G in 01:11:51 with 0 errors on Mon Sep 30 14:32:57 2024
config:

        NAME                        STATE     READ WRITE CKSUM
        zfs500mirror                ONLINE       0     0     0
          ata-ST3500312CS_9VV0PREA  ONLINE       0     0     0

errors: No known data errors
```



Now I needed to add it back using the attach command.  For this command, you need to add add the first drive that was healthy.  Then you list the second drive (new drive) you want to add into the mirror

zpool attach "Pool Name" /dev/disk/by-id/"Disk that was not changed" /dev/disk/by-id/"the new disk you added
```
root@pve1:~# zpool attach zfs500mirror /dev/disk/by-id/ata-ST3500312CS_9VV0PREA /dev/disk/by-id/ata-ST3500312CS_9VVPP2ZH
```


Now you will see it to start resilvering again
```
root@pve1:~# zpool status
  pool: zfs500mirror
 state: ONLINE
status: One or more devices is currently being resilvered.  The pool will
        continue to function, possibly in a degraded state.
action: Wait for the resilver to complete.
  scan: resilver in progress since Mon Sep 30 15:11:28 2024
        82.4G scanned at 107M/s, 11.3G issued at 14.6M/s, 82.4G total
        11.6G resilvered, 13.71% done, 01:23:06 to go
config:

        NAME                          STATE     READ WRITE CKSUM
        zfs500mirror                  ONLINE       0     0     0
          mirror-0                    ONLINE       0     0     0
            ata-ST3500312CS_9VV0PREA  ONLINE       0     0     0
            ata-ST3500312CS_9VVPP2ZH  ONLINE       0     0     0  (resilvering)

errors: No known data errors
```




After all the resilvering is done, its all done and healthy
```
root@pve1:~# zpool status zfs500mirror
  pool: zfs500mirror
 state: ONLINE
status: Some supported and requested features are not enabled on the pool.
        The pool can still be used, but some features are unavailable.
action: Enable all features using 'zpool upgrade'. Once this is done,
        the pool may no longer be accessible by software that does not support
        the features. See zpool-features(7) for details.
  scan: resilvered 84.0G in 01:11:51 with 0 errors on Mon Sep 30 16:23:19 2024
config:

        NAME                          STATE     READ WRITE CKSUM
        zfs500mirror                  ONLINE       0     0     0
          mirror-0                    ONLINE       0     0     0
            ata-ST3500312CS_9VV0PREA  ONLINE       0     0     0
            ata-ST3500312CS_9VVPP2ZH  ONLINE       0     0     0

errors: No known data errors
```








I watched this video
https://www.youtube.com/watch?v=IQA7aTezrVE