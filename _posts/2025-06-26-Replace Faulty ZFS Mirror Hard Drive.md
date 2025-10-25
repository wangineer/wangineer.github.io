---
title: How to Replace Mirror'd Hard Drive in ZFS Proxmox
description: Mirror Mirror on the Wall!
date: 2025-06-26 12:00:00:01 +/-0000
categories: [Proxmox, Troubleshoot]
tags: [server,raid,mirror,zfs,zpool]     # TAG names should always be lowercase
toc: true
comments: true
image:
  path: '/assets/img/headers/proxmoxraid1.webp'
  alt: 'Two hard drives showing connected to Proxmox box in Raid 1'
pin: false
published: true
---
## Stats:
 - ZFS Mirror using two 500gb hard drives
 - Proxmox Version 7.4-16
 - This is not a boot drive (Not sure if that matters)

## Scenario:
If I remember correctly, my server had lost power for some reason. The next morning I started receiving emails stating proxmox found and couldn't recover some errors. This ended up putting the "mirror" in a degraded state.  The errors were only in the single digits and not incrementing, so I wasn't terribly worried, but I went ahead and purchased a replcement drive.


## Pre-steps:
When you first experience this issue, you can always issue the below commands. And wait for the next scan to complete and see if more errors are found.
```
zpool status
zpool clear
zpool status
```

If you don't do the "zpool clear", you still get daily emails stating proxmox found uncorrectable errors.  I let this go one for many months. Oddly enough there was another power outage, and Proxmox stopped finding uncorrectable errors.  

Depending on how willing you are to wait for the errors to get bad or have a drive failure, you should probably start the below steps. 

1. Power off the server and pull out the faulty hard drive
  * When I first installed the hard drives in my server, I always made a note on my excel sheet what Hard drive/Serial Number/Brand/Size I installed
2. Ordered the Correct hard drive
3. Put the hard drive back in and powered the server back on.  
  * The drive is still part of the mirror and providing some tolerance of loss data. In the event the other drive happened to have a catastrophic failure, you wouldn't be totally out the data.
  * This is a good time to tell you that using a "mirror" as a form of data backup is not a good design.  I recommend having a seperate physical device/location to consider it a true backup.  Many things can occur to a mirror that can render your data unreadable. This could be something physical or, as simple as issuing the wrong command. 


## The Actual Repalcement

After many months of ignoring the emails, our house had another power outage. (One would really think that I should get a backup battery) After the device came onine, I saw a significant jump in error. 

Below command "zpool status"  shows that the hard drive with serial number 9VV0PEMS is in a Faulty State.

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



### Identify the Drive via CLI
I feel like I researched a lot on how to replace the drives on a proxmox mirror.  Having never done this before, I wanted to be as prepared as I could. I ended up finding a youtube video and gathering a collection of forums to develop a plan.  

### Identify the faulty Drive via CLI

It may be a good idea to get the below output
```
ls -ahlp /dev/disk/by-id/ 
```

## Power off/Pull Out/Push In/Power On
This next step we need to remove the drive so the server unmounts the drive.  I'm there there may be better ways of doin this, but this seemed to emulate a drive failure and provided the outputs needed to proceed forward. 

1. Power off the device (This can be done from the Proxmox GUI)
2. Once powered off, pull out the faulty hard drive.
3. With the faulty hard drive removed, put the new drive in.
4. Once the new drive is in the same slot, you can power on the server. 

Doing the above steps force the drive to be seen as Faulted and not mount on bootup.  You can see in the "zpool status" below that the faulty drive is in a degraded state and provides you the location id. "was dev/disk/by-id/" output.  At this point you don't see the new drive because it is not in the mirror yet.   

<b>You need to copy and save this from the "zpool status" it also provides you the "UUID"</b> 

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

### Identify the new Hard drive
To identify the new drive, we need to issue the command below by including the serial number. 

"ls -ahlp /dev/disk/by-id/grep #########"  <--- #### equals the Serial number of new drive

NOTE: "grep" is similar to using the "include" command in Cisco CLI. 

```
root@pve1:~# ls -ahlp /dev/disk/by-id/ |grep 9VVPP2ZH
lrwxrwxrwx 1 root root    9 Sep 30 15:11 ata-ST3500312CS_9VVPP2ZH -> ../../sdb
lrwxrwxrwx 1 root root   10 Sep 30 15:11 ata-ST3500312CS_9VVPP2ZH-part1 -> ../../sdb1
lrwxrwxrwx 1 root root   10 Sep 30 15:11 ata-ST3500312CS_9VVPP2ZH-part9 -> ../../sdb9
```

You can see that the Serialn umber ID is listed with the "ata-ST3500312CS_9VVPP2ZH" and the UUID "sdb" is at the end.


### Difference in steps
To swap drives in the mirror you will need to issue a "zpool replace" command. To do this, we need to reference the old location and the new location.  

The video I watched referenced the UUID... (sdb). This isn't recommended, as I'm told if the server reboots, then that UUID could possibly chnage.  

By watching the video, they stated to complete the below command after having identified location of hard drive. 

OLD ID: ata-ST3500312CS_9VV0PEMS-part1
NEW ID: ata-ST3500312CS_9VVPP2ZH
NEW UUID:sdb


Structure:
zpool replace "Pool Name" /dev/disk/by-id/"name you copied above was" "New location of harddrive

You can see below ouptut references the "UUID".  I later fix this later.
```
zpool replace zfs500mirror /dev/disk/by-id/ata-ST3500312CS_9VV0PEMS-part1 /dev/sdb
```

### Resilvering
Once I completed I did zpool status and saw it was replacing and resilvering
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


You can see the above name doesn't match the "ata" name.  It references the UUID instead.

### Completed Resilver and Healthy Status
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



## How to fix Mirror Reference from UUID to ID
If you want to replace this you need to complete a detach and then attach. 

### Detach
In the "detach" you referecne the Zpool mirror.  In my case zfs500mirror.  Then you reference the name you want removed from the mirror. 
```
root@pve1:~# zpool detach zfs500mirror sdb
```


You can now see the "sdb" removed from the mirror, and there is a single drive in the mirror.
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


### Attach
Now I needed to add it back using the attach command.  For this command, you need to add the first drive that was healthy.  Then you list the second drive (new drive) you want to add into the mirror

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



## Complete
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




# Conclusion 
Now that I've completed a Mirror replacement, I may start looking at two more projects

1. Moving my Boot Drive to the CD Tray
  #If I get that to work, I may also look at figuring out how to mirror the boot drive with a slot in back of my Dell R710.
2. My 8TB Mirror is almost full and I have two scenarios with the 8 Bay Server that i have. 
  * Buy two new hard drives of greater space and create a new mirror
  or
  * Buy a single hard drive of bigger size and replce one of the 8tb hard drives with the steps listed above. Once that is complete, Buy another hard drive down the road and replace the last 8tb hard drive when I really need the extra space.  Once replaced, I think there is a command to increase the space of Mirror.  
  * Buy a single hard drive, and create a new Mirror.  Copy all of the contents from the Old mirror to the new mirror, and wait until I have enough money to buy a new Hard drive of same space and add it to the new mirror. 

Definitely some fun options ahead. But it may take me a year to get there. 



# Resources
1. I watched this video
<https://www.youtube.com/watch?v=IQA7aTezrVE>

{% include embed/youtube.html id='IQA7aTezrVE' %}

2. Helped me understand the ID and UUID section
<https://dannyda.com/2020/09/24/how-to-add-attach-new-disk-to-existing-zfs-pool-on-proxmox-ve-pve-how-to-remove-detach-disk-from-zfs-pool-on-proxmox-ve/#google_vignette>

3. Future resource on how to replace Boot Drive
<https://theorangeone.net/posts/proxmox-boot-drive-replacement/>
