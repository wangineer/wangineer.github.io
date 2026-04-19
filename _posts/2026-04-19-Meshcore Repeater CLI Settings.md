---
title: MeshCore Repeater Advanced Tips (CLI) (RAK WisBlock)
description: A list of a few Meschore Repeater settings I used on the RAK WisBlock starter kit and 1W Booster, including OTA fix, RX gain tuning, AGC optimization, and radio performance tweaks.
date: 2026-04-17 00:00:00 +/-0000
categories: [offgrid, lora]
tags: [meshcore, rakwireless, wisblock, lora, repeater, 1w]
toc: true
comments: true
pin: false
published: true
permalink: /homelab/meshcore-repeater-rak-wisblock-settings
image:
  path: '/assets/img/headers/meshcore.webp'
seo:
  title: "MeshCore Repeater Advanced Tips (CLI) (RAK WisBlock)"
  description: "A list of a few Meschore Repeater settings I used on the RAK WisBlock starter kit and 1W Booster, including OTA fix, RX gain tuning, AGC optimization, and radio performance tweaks."
  canonical: "https://wangineer.com/homelab/meshcore-repeater-rak-wisblock-settings/"
  keywords: "meshcore repeater setup, rak wisblock lora, meshtastic repeater config, lora radio tuning, sx126x gain settings, rf mesh networking"
---

## MeshCore Repeater Setup (RAK WisBlock)

I recently set up a few MeshCore repeaters using some RAK WisBlock starter kits. 

This post shares a few of the configurations I applied across all of the repeaters to ensure repeatablility.

---

## Hardware Used

### Boards
- [RAK WisBlock Starter Kit (RAK4631)](https://amzn.to/4etK6uC)*
- [RAK 1W Booster](https://store.rakwireless.com/products/meshtastic-1w-lora-booster-kit-rak3401?srsltid=AfmBOopKlzZhE7vclP5BesbTbIo2xV0_icCDnCJA3-dQf4NmOGCbwAq6&variant=45678368882886) - I bought from AliExpress

> Make sure to purchase the 915mhz model if in the United States.
{: .prompt-warning }

### Antennas
- [External antenna (5.8 dBi recommended for attic or elevated installs)](https://store.rakwireless.com/products/5-8dbi-black-fiberglass-antenna?variant=44597332967622)
- [5dbi Alfa Antenna](https://muzi.works/products/alfa-outdoor-antenna?variant=44821986902073&country=US&currency=USD&utm_medium=product_sync&utm_source=google&utm_content=sag_organic&utm_campaign=sag_organic&srsltid=AfmBOoocQ-gDdmI_7bzV4mi44E3Aj5aOTtX7WP2iUIXa9kXAOUnTl4gKPUM)

> While at the Muzi Works site, I would buy below antennas for companions
{: .prompt-tip }



- [4 Pack Whip Antenna – 17cm 915MHz](https://muzi.works/products/4-pack-whip-antenna-17cm-915mhz)

### Power
- [Wisblock Power - USB-C Power Supply 5V](https://amzn.to/48Ds1Xg)*
- [Wisblock 1w Booster Power](https://amzn.to/4vDnhuw)*
- [Battery 10,000mah](https://www.makerfocus.com/products/makerfocus-3-7v-lipo-battery-10000mah-lithium-rechargeable-battery-9065115-with-micro-ph2-0-plug-for-raspberry-pi-ups-board?srsltid=AfmBOoqfjqSVkog2x_Qjh4hrdZHhAqZc5Q79F1wW9dxHrCMnviXdsmLo) - I bought this on Amazon, but not offered anymore.

### All inclusive
- [PeakMesh Altitude Node (Tree)](
https://www.etsy.com/listing/4331277320/peakmesh-altitude-tree-hanging-solar?ls=a&ga_order=most_relevant&ga_search_type=all&ga_view_type=gallery&ga_search_query=meshcore+solar&ref=sc_gallery-1-1&sr_prefetch=0&pf_from=market&frs=1&sts=1&plkey=Eu-SGt3Zlr4kIGUarPVNyP1qQE0b%3ALT91c79d395e4c86eb309a5f6d841292e2c0ffc42a&variation0=5555035768) - You can request in your checkout to have Meshcore Flashed on the device, otherwise you can do it yourself.

---

## Advanced CLI commands I used to manage my Rak Wisblock repeaters. 

### 1. Install OTA Fix (Highly Recommended)

Before doing anything else, install the OTA fix bootloader.

#### Why this matters
Without this fix, failed OTA updates can require physically accessing the device and reflashing over USB.  
With OTAfix, failed updates fall back to **DFU mode**, allowing recovery without tearing apart your setup.

#### Resource
https://github.com/oltaco/Adafruit_nRF52_Bootloader_OTAFIX

#### How To Install

1. Put your device into **DFU mode**
2. Drag and drop the `.uf2` file onto the device

Direct download:
https://github.com/oltaco/Adafruit_nRF52_Bootloader_OTAFIX/releases/download/0.9.2-OTAFIX2.1-BP1.2/update-wiscore_rak4631_board_bootloader-0.9.2-OTAFIX2.1-BP1.2_nosd.uf2

---

### 2. Enable RX Gain (If Upgrading from Older Firmware)

If you're running a version **prior to 1.14.1**, RX gain may not be enabled by default.

#### Why this matters
Enabling RX gain improves the receiver’s sensitivity, allowing your repeater to hear weaker signals — critical for range.

```bash
# get radio.rxgain
# set radio.rxgain on
```

---

### 3. Configure AGC Reset Interval

Automatic Gain Control (AGC) tuning helps stabilize signal handling.

#### Recommended values
- 4 (more responsive)
- 8 (balanced)
- 16 (more stable, less reactive)

```bash
# get agc.reset.interval
# set agc.reset.interval 4
```

#### Why this matters
Lower values react faster to changing RF conditions.  
Higher values provide smoother behavior in stable environments.

---

### 4. Set Path Hash Mode

```bash
# get path.hash.mode
# set path.hash.mode 1
```

#### Why this matters
Path hashing improves how routes are tracked across the mesh, helping reduce redundant transmissions and improving efficiency.

---

### 5. (Optional) Enable SX126X Boost Gain

> Use this cautiously depending on your RF environment
{: .prompt-warning }

```bash
# set lora.sx126x_rx_boosted_gain true
# reboot
```

#### Why this matters
This boosts receiver sensitivity even further, helping pick up very weak signals.

#### When to use it
- Good for **rural or low-interference environments**
- Not ideal for **dense RF areas (urban/suburban homes)**

Too much gain in noisy environments can actually reduce performance due to signal distortion.

---

## Practical Notes from My Setup

- My repeater is installed in an **attic**, which provides elevation without full outdoor exposure
- The **1W booster significantly improves uplink strength**, but receive tuning matters just as much
- Stability matters more than raw power — avoid over-tuning if your environment is noisy

---

## Recommended Baseline Configuration

If you just want a solid starting point:

```bash
# set radio.rxgain on
# set agc.reset.interval 4
# set path.hash.mode 1
```

Then test performance before enabling boosted gain.

---

## Conclusion

Setting up a MeshCore repeater isn’t just about plugging in hardware and turning it on. The real performance gains come from understanding how the radio behaves and making small, intentional adjustments.

The combination of OTA safety, RX tuning, and AGC optimization creates a node that is not only powerful but reliable over time. Once deployed in a good location, a properly tuned repeater can quietly improve the entire mesh without constant maintenance.

Start with the baseline settings, observe how your node behaves, and make incremental adjustments. Over time, you’ll find the balance that works best for your environment.

---

## Call to Action

If you’re running a similar setup, try these settings and compare performance before and after.

If you notice improvements (or issues), drop a comment — I’d be interested to see how different environments impact tuning.


* This blog post uses Affiliate links to Amazon. 