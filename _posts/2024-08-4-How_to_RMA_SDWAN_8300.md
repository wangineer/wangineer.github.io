---
title: How to replace a Cisco SDWAN 8300
description: They don't build 'em like they used to!
date: 2024-08-04 20:00:00:01 +/-0000
categories: [NetworkEngineer, RMA]
tags: [SDWAN,RMA,c8300,cisco]     # TAG names should always be lowercase
toc: true
comments: active
ping: true
image:
  path: '/assets/img/headers/8300.webp'
pin: false
published: true
---

## The Story

Back in the day replacing a faulty hardware was fairly simple.  As long as you had console and or management access to the new hardware, you could get up and going pretty easily.  This not the case with Cisco SDWAN devices in Controller Mode using CLI.  

## Pre-Steps

1. Register device Serial number in Cisco PnP Portal
2. Download PnP vbond file to yoru pc (preferrably device that has FTP access)
   * If your device does not have access to ftp, you can also try downloading to USB device
3. Upload file to SDWAN Portal
 * vManage -> Configurations -> Devices -> Select 'Upload WAN Edge list'
 * Click Browse and select the file (serialFile.viptella)
 * Check the box - Validate the uploaded vEdge List and send to controllers
 * Click Upload
4. Generate BootStrap file
 * Log into FTP server (or wherever you plan to get the file and transport to your hardware)
 * Locate device
     * vManage -> Configurations -> Devices > Search for the Serial# atached to your RMA device 
  * Click Three dots on the right of th edevice and select:
     * Generate Bootstrap Configurations
     * Choose Defaults (ok)
     * Download the File and rename it "ciscosdwan.cfg"
5. Copy previously downloaded bootstrap file to new RMA device with either USB or FTP
YOU WILL NEED TO MODIFY THE FILE LIKE REMOVE TACACS AND POSSIBLY FEW OTHER THINGS
6. Based on which mode the device is in issue the below command.
  * Autonomous Mode: controller-mode enable
  * Controller Mode: request platform software sdwan config reset
7. Log into device : After device comes online default username/password
     admin/admin
8. Validate certificates are applied
   * show sdwan control local-properties

9. Pre-Stage "show sdwan run"
   * Log into active device and copy all output from "show sdwan run" and paste into a notepad

10. Modify the following output
    * Remove "username admin..."
    * Remove interface ........ and sub interfaces.... (If you don't you will have duplicate IP's on the same network)
    * Verify Default route is in the configuration
    * Remove Tacacs configurations
      * aaa group server tacacs+ "tacacs_name"  # don't use the quotes

11. Copy entire configuration from Notepad and paste it into the RMA device.
    * config-transaction
      paste entire notepad file

If you have errors don't commit the changes.  exit it out and fix the errors

12. Re-apply entire Notepad file again and there should be no more errors. so you can commit at the end
    * commit

13. Compare "show run" and "show sdwan run" between the two devices

14. Log into SDWAN portal and verify certificates are valid for the RMA device

15. Complete following outputs on active device
   * show version
   * show platform
   * show cdp neighbors
   * show ip int brief
   * show ip protocols
   * show interface | inc CRC|errors
   * show ip bgp all summary
   * show arp
   * show sdwan control connections
   * show sdwan bfd sessions
   * show sdwan bfd sessions | inc down
   * show license udi
   * show license summary
   * show license summary all
   * show ip route | show ip route vrf "list vrf"


## Implementation Plan

1. Do the things. 
... to be continued. 