---
title: How to Factory Reset an IOS-XR Router
description: Can't Write Erase anymore
date: 2024-10-24 12:00:00:01 +/-0000
categories: [NetworkEngineer, Decommission]
tags: [iosxr,factory-reset,cisco,]     # TAG names should always be lowercase
toc: true
comments: active
ping: true
image:
  path: '/assets/img/headers/cisco.webp'
pin: false
published: true
---
## Scenario
There comes a time in life where a Network Engineer needs to turn the lights and decommission a router or switch.  This can be caused due to a Hardware upgrade, or you need to replace the device because it has hit some faulty bug.  Its never a good idea to power down a device without cleaning up your configurations.  

With Cisco the commands we used to perform the below commands.
 - write erase
 - delete vlan.dat
 - reload

With Newer IOS versioning, we can now complete a 3-pass cleaning of the hardware.  This is like Department of Defense wiping of your hardware.  The problem sometimes though, if you want to reuse the hardware later, it additionally removes any firmare files and removes the boot variable.  So if you turn it back on, it will go directly to ROMMON mode.  Which isn't a problem, just a few extra steps in trying to get it back online.  

## Disclaimer
Please note that by following any commands provided, I am not liable for any destruction or unexpected behavior your your network or environment.  I'm not sure if this is legally binding, but this is all for informational purposes and what worked for me. 

## Research
When I initially tried researching how to factory reset a device, I couldn't quite find the right documentation.  I'm sure there is more documentation now on it, but i happened to save my session so that I could go back and remember the simple commands. 

## The Commands to Factory Reset
Note: I would recommend doing this via Console if you have the option.  When you complete the output all interfaces will go down and will become unreachable

```
RP/0/RP0/CPU0:Router_Hostname#conf t


Mon Jul  3 18:38:35.361 UTC

RP/0/RP0/CPU0:Router_Hostname(config)#commit replace


Mon Jul  3 18:39:27.995 UTC



This commit will replace or remove the entire running configuration. This

operation can be service affecting.

Do you wish to proceed? [no]: yes
```

After you type "yes" you will see a few logs about Smart licensing communication failing, as well as interfaces and optics starting to go down.  
Once interface down logs stop, and you see the hostname change to "ios" you can try exiting out of the config mode and complete a "show running" to see if it has been wiped.  

```
RP/0/RP0/CPU0:ios(config)#exit

RP/0/RP0/CPU0:ios#show run


Mon Jul  3 18:41:27.089 UTC

Building configuration...

!! IOS XR Configuration 7.2.2

!! Last configuration change at Mon Jul  3 18:39:47 2023 by (USERNAME)

!

end
```

Now for good measure I also like to reload the device.  When I completed the generic "reload" command i got an error message

```
RP/0/RP0/CPU0:ios#reload


Mon Jul  3 18:43:07.878 UTC

WARNING: Standby card not present or not ready yet. Reload command will reload only RP VM and it leads to unexpected behavior. Use 'reload location all' to properly reload the box.
```

So I went ahead and completed the "reload location all"

```
RP/0/RP0/CPU0:ios#reload location all
```

Below is the full output after completing the reload commands.

```
RP/0/RP0/CPU0:ios#reload location all


Mon Jul  3 18:43:44.601 UTC



Standby card not present or not Ready for failover. Proceed? [confirm]



Preparing system for backup. This may take a few minutes especially for large configurations.

	Status report: node0_RP0_CPU0: START TO BACKUP 

	Status report: node0_RP0_CPU0: BACKUP HAS COMPLETED SUCCESSFULLY 

[Done]



Proceed with reload?  [confirm]%SMART_LIC-3-COMM_FAILED:Communications failure with the Cisco Smart Software Manager (CSSM) : The action failed because smart license transport is disabled in call-home. Please retry after manually enabling smart license transport with command "reporting smart-licensing-data" under the profile you want to send out Smart Licensing data, and also make sure the profile is active. 
%SMART_LIC-3-AUTH_RENEW_FAILED:Authorization renewal with the Cisco Smart Software Manager (CSSM) : Communication message send error for udi PID:NCS-55A2-MOD-S,SN:FOC2311R1HE


Reloading node  all 

RL: Reboot initiated with code 1, cause User initiated graceful reload reboot_timeout 30 shutdown delay 0

RL: Shutdown initiated

Query the node to be reloaded 

 IP of node to be reloaded 192.0.0.4

sending stop hb

Cause: User initiated graceful reload

VM IP addr sent for reload 192.0.0.4





Received ack from sdrmgr for reload request.Returncode:0

successful disconnection from service 

wd_disconnect_cb 550 CMP-WD disconnected successfully

Invmgr successful disconnection from service

RP/0/RP0/CPU0:ios#
Disconnecting from 'default-sdr--1' console. Continue(Y/N)?


Connecting to 'default-sdr--1' console




This product contains cryptographic features and is subject to United 
States and local country laws governing import, export, transfer and 
use. Delivery of Cisco cryptographic products does not imply third-party 
authority to import, export, distribute or use encryption. Importers, 
exporters, distributors and users are responsible for compliance with 
U.S. and local country laws. By using this product you agree to comply 
with applicable laws and regulations. If you are unable to comply with 
U.S. and local laws, return this product immediately. 

A summary of U.S. laws governing Cisco cryptographic products may be 
found at:
http://www.cisco.com/wwl/export/crypto/tool/stqrg.html

If you require further assistance please contact us by sending email to 
export@cisco.com.



LC/0/0/CPU0:Jul  3 18:45:26.425 UTC: tam_entropy[324]: %SECURITY-TAMSVCS-3-ENTROPY : Shutting down tam_entropy process for Software TAM based devices. 



!!!!!!!!!!!!!!!!!!!! NO root-system username is configured. Need to configure root-system username. !!!!!!!!!!!!!!!!!!!!Configuration lock is held by another agent.  Please wait. [..OK]






         --- Administrative User Dialog ---





  Enter root-system username: LC/0/0/CPU0:Jul  3 18:45:50.005 UTC: optics_driver[164]: %PKT_INFRA-FM-3-FAULT_MAJOR : ALARM_MAJOR :OPTICS RX LOS LANE-0 :DECLARE :0/0/CPU0:  Optics0/0/0/4 

LC/0/0/CPU0:Jul  3 18:45:50.005 UTC: optics_driver[164]: %PKT_INFRA-FM-3-FAULT_MAJOR : ALARM_MAJOR :OPTICS RX POWER LANE-0 LOW ALARM :DECLARE :0/0/CPU0:  Optics0/0/0/4 

LC/0/0/CPU0:Jul  3 18:45:50.673 UTC: optics_driver[164]: %PKT_INFRA-FM-3-FAULT_MAJOR : ALARM_MAJOR :OPTICS RX LOS LANE-0 :DECLARE :0/0/CPU0:  Optics0/0/0/9 

LC/0/0/CPU0:Jul  3 18:45:50.673 UTC: optics_driver[164]: %PKT_INFRA-FM-3-FAULT_MAJOR : ALARM_MAJOR :OPTICS RX POWER LANE-0 LOW ALARM :DECLARE :0/0/CPU0:  Optics0/0/0/9 

LC/0/0/CPU0:Jul  3 18:45:51.341 UTC: optics_driver[164]: %PKT_INFRA-FM-3-FAULT_MAJOR : ALARM_MAJOR :OPTICS RX LOS LANE-0 :DECLARE :0/0/CPU0:  Optics0/0/0/10 

LC/0/0/CPU0:Jul  3 18:45:51.341 UTC: optics_driver[164]: %PKT_INFRA-FM-3-FAULT_MAJOR : ALARM_MAJOR :OPTICS RX POWER LANE-0 LOW ALARM :DECLARE :0/0/CPU0:  Optics0/0/0/10 

LC/0/0/CPU0:Jul  3 18:46:12.028 UTC: optics_driver[164]: %PKT_INFRA-FM-3-FAULT_MAJOR : ALARM_MAJOR :OPTICS RX LOS LANE-0 :CLEAR :0/0/CPU0:  Optics0/0/0/4 

LC/0/0/CPU0:Jul  3 18:46:12.028 UTC: optics_driver[164]: %PKT_INFRA-FM-3-FAULT_MAJOR : ALARM_MAJOR :OPTICS RX POWER LANE-0 LOW ALARM :CLEAR :0/0/CPU0:  Optics0/0/0/4 

LC/0/0/CPU0:Jul  3 18:46:12.695 UTC: optics_driver[164]: %PKT_INFRA-FM-3-FAULT_MAJOR : ALARM_MAJOR :OPTICS RX LOS LANE-0 :CLEAR :0/0/CPU0:  Optics0/0/0/9 

LC/0/0/CPU0:Jul  3 18:46:12.695 UTC: optics_driver[164]: %PKT_INFRA-FM-3-FAULT_MAJOR : ALARM_MAJOR :OPTICS RX POWER LANE-0 LOW ALARM :CLEAR :0/0/CPU0:  Optics0/0/0/9 

LC/0/0/CPU0:Jul  3 18:46:13.362 UTC: optics_driver[164]: %PKT_INFRA-FM-3-FAULT_MAJOR : ALARM_MAJOR :OPTICS RX LOS LANE-0 :CLEAR :0/0/CPU0:  Optics0/0/0/10 

LC/0/0/CPU0:Jul  3 18:46:13.362 UTC: optics_driver[164]: %PKT_INFRA-FM-3-FAULT_MAJOR : ALARM_MAJOR :OPTICS RX POWER LANE-0 LOW ALARM :CLEAR :0/0/CPU0:  Optics0/0/0/10 





  % Entry must not be null.



  Enter root-system username: %SMART_LIC-3-COMM_FAILED:Communications failure with the Cisco Smart Software Manager (CSSM) : Fail to send out Call Home HTTP message 
%SMART_LIC-3-AUTH_RENEW_FAILED:Authorization renewal with the Cisco Smart Software Manager (CSSM) : Communication message send error for udi PID:NCS-55A2-MOD-S,SN:FOC2311R1HE




  % Entry must not be null.



  Enter root-system username:
  ```

  When I saw the username prompt i stopped. 



## Conclusion 
Well, there you have it.  IOS-XR devices are a bit different when it comes to there command structure.  I believe they improved there process with the Viptella/SDWAN controller mode devices running the new 17.x.x code.  
Hope this helps anybody if they happen to need to wipe a device.  

## Resources
1. I found the below Cisco Community forum today to backup findings since I knew the actual command
  - <https://community.cisco.com/t5/networking-knowledge-base/quot-commit-replace-quot-command-in-cisco-ios-xr/ta-p/3153954>

2. Cisco Documentation for "replace"
  - <https://www.cisco.com/c/en/us/td/docs/routers/xr12000/software/xr12k_r3-9/system_management/command/reference/yr39xr12k_chapter5.html>
3. This documentation goes into actually clearing out user data
  - <https://community.cisco.com/t5/service-providers-knowledge-base/factory-reset-erasing-wiping-out-user-data-from-disk-memory-on/ta-p/4565678>
