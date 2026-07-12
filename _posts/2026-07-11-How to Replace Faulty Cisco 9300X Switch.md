---
layout: post
title: "Replacing a Faulty Cisco Catalyst 9300X: Prestaging and Cable Order"
date: 2026-07-10 08:00:00 -0700
categories: [Networking, Cisco]
tags: [Cisco, Catalyst 9300X, StackWise, StackPower, IOS XE, Troubleshooting, CCNP]
description: "A practical walkthrough for replacing a failed Cisco Catalyst 9300X stack member, with emphasis on prestaging, StackWise speed, and the safe order for disconnecting and reconnecting power, StackPower, and StackWise cables."
permalink: /posts/replace-cisco-catalyst-9300x-stack-member/
image:
  path: /assets/img/headers/cisco-9300x-stack-member-replacement.webp
  alt: "Cisco Catalyst 9300X switch being added to a StackWise stack"
seo:
  title: "Replace a Cisco Catalyst 9300X Stack Member Safely"
  description: "Learn how to prestage a Cisco Catalyst 9300X replacement and use the correct power, StackPower, and StackWise cable order to avoid a stack merge or unexpected reload."
  canonical: /posts/replace-cisco-catalyst-9300x-stack-member/
  keywords: "Cisco 9300X replacement, Catalyst 9300X StackWise, StackPower cable order, replace stack member, stack merge, switch stack-speed high, IOS XE auto upgrade"
---

Replacing a failed switch in a Catalyst stack sounds straightforward: rack the replacement, move the cables, and power it on.

Two details made this replacement less routine:

1. The replacement Catalyst 9300X had to use the same StackWise ring speed as the existing stack.
2. The power, StackPower, and StackWise cables had to be removed and installed in the correct order.

The StackWise speed mismatch prevented the replacement from joining the existing 1 Tbps stack. The replacement was still configured for the lower 480 Gbps mode and came up as its own active switch.

The cable order presented the larger operational risk. A switch connected to StackPower may still be receiving power even after its normal AC power cords are removed. If both StackWise data cables are removed while that switch is still powered, it can become an isolated standalone stack. Reconnecting that powered switch to the production stack can trigger a stack merge, an election, and an unexpected reload of the stack.

This post documents both the prestaging work and the physical cable sequence I would use the next time.

> This is a sanitized example. It assumes an identical Catalyst 9300X is replacing failed member 3 in an existing high speed StackWise ring. Confirm the exact model, supported IOS XE release, licensing, boot mode, and change procedure for your environment.
{: .prompt-info }

## Why prestaging matters

A replacement member should match the existing stack in several areas:

- Hardware model and supported stack type
- IOS XE version
- Install or bundle boot mode
- License level
- StackWise ring speed
- Stack member number
- Stack priority
- StackWise data cable layout
- StackPower topology and available power budget
- The order in which power, StackPower, and StackWise cables are moved

A mismatch can leave the replacement in a `V-Mismatch` state, create a separate active island, or split the stack into subrings. An incorrect cable sequence can also create a powered standalone switch that later causes a stack merge when it is reconnected.

## 1. Record the state of the working stack

Before removing hardware, collect enough information to recreate the failed member and validate the stack afterward.

```console
show switch
show switch detail
show switch stack-ring speed
show switch stack-bandwidth
show switch stack-ports summary
show stack-power detail
show stack-power budgeting
show stack-power neighbors
show environment power all
show version
show boot
show install summary
show license summary
show inventory
show running-config | section ^switch
```

Example:

```text
C9300X-STACK# show switch

Switch/Stack Mac Address : 00aa.11bb.22cc
Mac persistency wait time: Indefinite

                                           H/W   Current
Switch#   Role      Mac Address     Priority Version State
------------------------------------------------------------
 1        Active    00aa.11bb.2200     15     V02     Ready
 2        Standby   00aa.11bb.2300     14     V02     Ready
 3        Member    0000.0000.0000      0              Removed
```

The failed member number is important because interface configuration is tied to that number. An identical replacement that joins as member 3 can inherit the existing configuration for interfaces such as `TenGigabitEthernet3/0/1`.

Before disconnecting any power source, also record the StackPower topology and budget:

```text
C9300X-STACK# show stack-power detail

Power Stack                      Stack   Stack    Total   Rsvd    Alloc   Sw_Avail
Name                             Mode    Topology Pwr(W)  Pwr(W)  Pwr(W) Pwr(W)
---------------------------------------------------------------------------------
Powerstack-1                     SP-PS   Ring     4290    30      1880   2380

Switch 3:
  Port 1 status: Connected
  Port 2 status: Connected
  Neighbor on port 1: Switch 2
  Neighbor on port 2: Switch 1
```

This confirms that the failed member is participating in a StackPower ring. It also helps determine whether removing its local power supplies or StackPower links could reduce the available power enough to cause PoE load shedding or power down another member.

> Do not treat StackPower as a passive cable. It carries power. A switch can remain powered through StackPower after both local AC power cords are removed.
{: .prompt-warning }

Check the current and next boot StackWise speed:

```text
C9300X-STACK# show switch stack-ring speed

Stack Ring Speed          : 1000G
Stack Ring Configuration  : Full
Stack Ring Protocol       : StackWise
Stack Ring Next-boot Speed: 1000G
```

Also confirm the boot mode and software version:

```text
C9300X-STACK# show boot

Current Boot Variables:
BOOT variable = bootflash:packages.conf;

Boot Variables on next reload:
BOOT variable = bootflash:packages.conf;
Manual Boot = no
```

```text
C9300X-STACK# show install summary

[ Switch 1 2 ]
Installed Package(s) Information:
State (St): I = Inactive, U = Activated and Uncommitted,
            C = Activated and Committed

Type  St   Filename/Version
-----------------------------------------------
IMG   C    17.12.04
```

## 2. Console into the replacement switch

Do not connect the replacement to the production stack yet.

Power it on as a standalone switch and confirm what was received.

```console
show inventory
show switch
show version
show boot
show install summary
show license summary
show switch stack-ring speed
```

A factory or previously used switch may show something similar to this:

```text
Replacement# show switch

                                           H/W   Current
Switch#   Role      Mac Address     Priority Version State
------------------------------------------------------------
*1        Active    00dd.44ee.5500      1     V02     Ready
```

```text
Replacement# show switch stack-ring speed

Stack Ring Speed          : 480G
Stack Ring Configuration  : Standalone
Stack Ring Protocol       : StackWise
Stack Ring Next-boot Speed: 480G
```

That 480G value is the problem when the destination stack is running at 1000G.

## 3. Choose how to align the IOS XE software

The replacement must use compatible software and the same operational mode as the stack.

### Option A: Manually upgrade or downgrade the replacement

Copy the required IOS XE image to flash, then use the install workflow.

```console
install add file flash:cat9k_iosxe.<TARGET_VERSION>.SPA.bin activate commit
```

Example:

```text
Replacement# install add file flash:cat9k_iosxe.17.12.04.SPA.bin activate commit

install_add_activate_commit: START
Checking whether new add is allowed...
Package added successfully.
Activating package...
This operation requires a reload of the system.
Do you want to proceed? [y/n] y
```

The switch reloads during activation. After it returns, validate that the image is committed and that it boots from `packages.conf`.

```console
show version
show boot
show install summary
```

The same install command can be used for a supported downgrade, but the target release and downgrade path should be checked against the release notes first.

### Option B: Let the existing stack perform the auto upgrade

On the active switch, enable software auto upgrade before adding the replacement:

```console
configure terminal
 software auto-upgrade enable
end
write memory
```

Software auto upgrade is available when the stack is running in Install mode. The replacement may reload while the stack synchronizes its software.

I still prefer to manually align the image during prestaging when possible. It makes the maintenance window more predictable and reduces the amount of work that must happen after the new switch is physically installed.

## 4. Match the StackWise ring speed

For an all C9300X high speed stack, configure the replacement for high speed mode.

```console
configure terminal
 switch stack-speed high
end
write memory
```

The setting does not take effect until the switch reloads.

```text
Replacement# show switch stack-ring speed

Stack Ring Speed          : 480G
Stack Ring Configuration  : Standalone
Stack Ring Protocol       : StackWise
Stack Ring Next-boot Speed: 1000G
```

> Use `switch stack-speed low` when the destination stack is operating at 480 Gbps. A mixed stack or a stack containing hardware that does not support high speed mode must use the supported lower speed.
{: .prompt-warning }

## 5. Set the replacement member number

A standalone switch normally starts as member 1. In this example, the failed switch was member 3.

```console
switch 1 renumber 3
```

Example prompt:

```text
Replacement# switch 1 renumber 3

WARNING: Changing the switch number may result in a configuration change.
The interface configuration associated with the old switch number will
remain as a provisioned configuration.
The new switch number will become effective after the next reload.
Do you want to continue? [y/n] y
```

Save the configuration and reload. This reload can apply both the new member number and the new StackWise speed.

```console
write memory
reload
```

### Expect a leftover provisioned member

After the switch returns, the physical switch should be member 3. However, `show switch` may also display the old member 1 as `Provisioned`.

```text
Replacement# show switch

                                           H/W   Current
Switch#   Role         Mac Address     Priority Version State
---------------------------------------------------------------
 1        Provisioned  0000.0000.0000      0     V02     Provisioned
*3        Active       00dd.44ee.5500      1     V02     Ready
```

This is not a second physical switch. The renumber operation retained the old member 1 configuration as an offline provisioned entry.

Confirm the provision statements in the running configuration:

```console
show running-config | include ^switch.*provision
```

Example:

```text
Replacement# show running-config | include ^switch.*provision
switch 1 provision c9300x-48hx
switch 3 provision c9300x-48hx
```

Before removing member 1, confirm all of the following:

- The physical replacement is now member 3 and is in the `Ready` state
- Member 1 has a zero MAC address and is only `Provisioned`
- No member 1 interface configuration needs to be retained
- The destination production stack expects the replacement to join as member 3

> The `no switch 1 provision` command removes the provision statement and configuration associated with member 1, including any member 1 interface configuration. Do not run it against a member number that still represents a physical switch or configuration you need to preserve.
{: .prompt-warning }

Remove the stale member 1 provisioned entry:

```console
configure terminal
 no switch 1 provision
end
write memory
reload
```

In the behavior that prompted this post, the cleanup required another reload before the stale member completely disappeared from the stack table. Plan for this additional prestaging reload rather than discovering it during the maintenance window.

After the second reload, verify that only member 3 remains and that the high speed setting is active:

```text
Replacement# show switch

                                           H/W   Current
Switch#   Role      Mac Address     Priority Version State
------------------------------------------------------------
*3        Active    00dd.44ee.5500      1     V02     Ready
```

```text
Replacement# show running-config | include ^switch.*provision
switch 3 provision c9300x-48hx
```

```text
Replacement# show switch stack-ring speed

Stack Ring Speed          : 1000G
Stack Ring Configuration  : Standalone
Stack Ring Protocol       : StackWise
Stack Ring Next-boot Speed: 1000G
```

## 6. Set the stack priority

Priority influences future active and standby elections. It does not immediately replace the current active switch.

Use the priority that fits the stack design. Do not automatically assign 15 to every replacement.

In this example, members 1 and 2 are intended to remain active and standby, so member 3 receives a lower priority:

```console
switch 3 priority 5
write memory
```

Example:

```text
Replacement# switch 3 priority 5

Changing the Switch Priority of Switch Number 3 to 5
Do you want to continue? [confirm]
```

Verify:

```text
Replacement# show switch

                                           H/W   Current
Switch#   Role      Mac Address     Priority Version State
------------------------------------------------------------
*3        Active    00dd.44ee.5500      5     V02     Ready
```

It is active only because it is currently operating by itself.

## 7. Follow the cable sequence exactly

The physical replacement is not simply a matter of moving every cable from the old chassis to the new one.

There are three separate connection types involved:

| Connection | Purpose | Important behavior |
|---|---|---|
| AC or DC power cords | Provide local power to the switch power supplies | Removing them may not turn off a switch that is still receiving StackPower |
| StackPower cables | Share power between stack members | A connected StackPower cable can keep the chassis powered |
| StackWise cables | Carry the data stack control and forwarding traffic | Connecting a powered standalone switch can trigger a stack merge and reload |

The basic rule is:

> **Removal order:** local power, StackPower, confirm the switch is off, then StackWise.
>
> **Installation order:** StackWise first, then StackPower, then local power.
{: .prompt-danger }

### Before touching any cable

Validate the target member and the health of both stack rings:

```console
show switch
show redundancy
show switch stack-ports summary
show switch stack-ring speed
show stack-power detail
show stack-power budgeting
show stack-power neighbors
show environment power all
```

Confirm:

- The correct failed member has been identified
- Whether the failed member is active, standby, or a regular member
- The StackWise data ring is healthy before the change
- The StackPower topology and neighbor relationships are documented
- The remaining StackPower budget can support the stack when the failed member's local power supplies are removed
- Console access is available to the stack and the replacement

> This procedure assumes the failed switch is not the active member. Removing the active switch causes the standby to become active. That is a separate control-plane event and should be explicitly included in the change plan.
{: .prompt-info }

### Remove the failed switch

Use this order:

#### 1. Remove the local power cords

Disconnect every AC or DC input from the failed switch, including both power supplies when two are installed.

Do not assume the switch is off yet.

#### 2. Remove the StackPower cables

Disconnect the StackPower cables from the failed member according to the documented power-ring cable plan.

StackPower is a power source. The switch may remain fully operational after its local power cords are removed and may not turn off until the StackPower connections are also removed.

When modifying a live StackPower ring, avoid opening more of the power ring than the approved design requires. Cisco recommends that an operational StackPower ring be broken at only one point at a time to reduce the chance of service interruption. Confirm the available power budget before each change.

#### 3. Confirm the chassis is completely powered off

Before touching either StackWise data cable, verify:

- All switch LEDs are off
- Both power-supply LEDs are off
- The console no longer responds
- The stack reports the member as removed
- No StackPower connection is still supplying the chassis

```console
show switch
show stack-power detail
show stack-power neighbors
```

Example:

```text
C9300X-STACK# show switch

                                           H/W   Current
Switch#   Role      Mac Address     Priority Version State
------------------------------------------------------------
*1        Active    00aa.11bb.2200     15     V02     Ready
 2        Standby   00aa.11bb.2300     14     V02     Ready
 3        Member    0000.0000.0000      0              Removed
```

#### 4. Remove both StackWise data cables

Only after the switch is confirmed fully powered off should the StackWise data cables be disconnected from the failed member.

Label each cable before removal. Record which neighbor and StackWise port each cable connects to. Do not rely only on the physical position of the switches in the rack.

#### 5. Remove the front-panel network connections

Move or label access links, uplinks, port-channel members, console, management, and any other front-panel connections. The failed chassis can then be removed from the rack.

### Install the replacement switch

The replacement must remain completely unpowered while the StackWise data cables are connected.

That means:

- Local AC or DC power cords are disconnected
- Both StackPower cables are disconnected
- All LEDs are off
- The switch is not still powered from a prestaging connection

Use this order:

#### 1. Rack the replacement while it is unpowered

Confirm the serial number, model, member number, IOS XE version, install mode, license level, priority, and next-boot StackWise speed.

#### 2. Connect both StackWise data cables first

Reconnect the StackWise cables to the exact ports recorded during removal. Finger-tighten the connector screws and verify that the intended full-ring topology is restored.

Do not connect either power source before the StackWise data cables are in place.

#### 3. Reconnect the front-panel network connections

Move the access links, uplinks, port-channel members, management, and other network connections while the replacement is still unpowered.

This prevents interfaces from becoming active before the replacement has joined the stack and received the configuration associated with member 3.

#### 4. Connect the StackPower cables

Reconnect the StackPower cables according to the recorded topology.

> The replacement may begin booting as soon as the first live StackPower cable is attached. At this point, both StackWise data cables must already be connected.
{: .prompt-warning }

Monitor the console as soon as StackPower is connected.

#### 5. Connect the local power cords last

Connect the local AC or DC power inputs after the StackWise and StackPower cables are secured.

The replacement should boot, discover the active stack, and join as member 3.

### Why the reverse order matters

The dangerous sequence is:

1. Power on the replacement as a standalone switch
2. Allow it to boot as its own active switch
3. Connect its StackWise cables to the live production stack

That is a powered stack merge. Cisco documents that merging powered-on stacks causes the switches to reload and elect a new active member.

The safe sequence is:

```text
REMOVE:
Local power -> StackPower -> verify completely off -> StackWise

INSTALL:
StackWise -> network cables -> StackPower -> local power
```

The replacement must never be a powered standalone switch at the moment its StackWise data cables are connected to the production stack.

If the software, member number, StackWise speed, hardware, and cable sequence are correct, the replacement should join as member 3 and receive the configuration already associated with that member.

## 8. Validate the repaired stack

Start with `show switch detail`.

```text
C9300X-STACK# show switch detail

Switch/Stack Mac Address : 00aa.11bb.22cc
Mac persistency wait time: Indefinite

                                           H/W   Current
Switch#   Role      Mac Address     Priority Version State
------------------------------------------------------------
*1        Active    00aa.11bb.2200     15     V02     Ready
 2        Standby   00aa.11bb.2300     14     V02     Ready
 3        Member    00dd.44ee.5500      5     V02     Ready

Stack Port Status                 Neighbors
Switch#   Port 1   Port 2         Port 1   Port 2
--------------------------------------------------
1         OK       OK             2        3
2         OK       OK             3        1
3         OK       OK             1        2
```

Confirm that the ring is full and running at the expected speed:

```text
C9300X-STACK# show switch stack-ring speed

Stack Ring Speed          : 1000G
Stack Ring Configuration  : Full
Stack Ring Protocol       : StackWise
Stack Ring Next-boot Speed: 1000G
```

Check every stack port:

```text
C9300X-STACK# show switch stack-ports summary

Sw#/Port#  Port Status  Neighbor  Cable Length  Link OK  Link Active  Sync OK  #Changes to LinkOK  In Loopback
----------------------------------------------------------------------------------------------------------------
1/1        OK           2         50cm          Yes      Yes          Yes      1                   No
1/2        OK           3         50cm          Yes      Yes          Yes      1                   No
2/1        OK           3         50cm          Yes      Yes          Yes      1                   No
2/2        OK           1         50cm          Yes      Yes          Yes      1                   No
3/1        OK           1         50cm          Yes      Yes          Yes      1                   No
3/2        OK           2         50cm          Yes      Yes          Yes      1                   No
```

A rising `#Changes to LinkOK` counter can indicate a loose cable, damaged stack cable, or unstable stack port.

Confirm full bandwidth:

```text
C9300X-STACK# show switch stack-bandwidth

Stack Current
Switch#   Role      Bandwidth   State
--------------------------------------
*1        Active    1000G       Ready
 2        Standby   1000G       Ready
 3        Member    1000G       Ready
```

Finish with software, redundancy, inventory, interface, and log checks:

```console
show version
show boot
show install summary
show redundancy
show inventory
show interfaces status
show logging | include STACK|SWITCH|INSTALL|REDUNDANCY
```

Also validate the services connected to the replaced member:

- Expected access and trunk ports are up
- Port channels have the correct members
- VLANs and spanning tree state are correct
- Routing adjacencies are restored
- Power over Ethernet devices are receiving power
- Monitoring and configuration backup systems recognize the new serial number
- No unexpected errors are increasing on uplinks or stack ports

## Reload plan

This process may include several reloads:

1. One reload for a manual IOS XE upgrade or downgrade
2. One reload to apply the StackWise speed and member number
3. One cleanup reload after removing the stale old-member provision statement
4. A possible additional reload if software auto upgrade runs after the switch joins

Combining the stack speed and renumber changes before the same reload can reduce the prestaging time. The stale provisioned-member cleanup should still be treated as a separate validation and reload step.

## What I learned

The replacement hardware was not the difficult part. The two important lessons were:

1. **StackWise speed is part of compatibility.**
2. **The cable sequence determines whether the replacement joins safely or creates a powered stack merge.**

A C9300X configured for 480 Gbps cannot form a stack link with members operating at 1 Tbps. When the speeds do not match, the replacement may look healthy from the console but remain isolated as its own active switch.

Just as importantly, removing normal power cords does not guarantee that a StackPower-connected switch is off. The safe approach is to remove local power, remove StackPower, verify that the chassis is completely off, and only then remove the StackWise data cables. Installation is the reverse: StackWise first, followed by StackPower and local power.

The commands I would always capture before and after a replacement are:

```console
show switch detail
show running-config | include ^switch.*provision
show switch stack-ring speed
show switch stack-ports summary
show switch stack-bandwidth
show stack-power detail
show stack-power budgeting
show stack-power neighbors
show environment power all
show install summary
show boot
```

Those commands quickly answer the most important questions:

- Did the replacement join with the correct member number?
- Are active, standby, and member roles correct?
- Is the stack a full ring?
- Do the stack ports have valid neighbors?
- Is the data ring operating at the expected speed?
- Is the StackPower topology correct and adequately powered?
- Is every member running the committed IOS XE package?
- Was the replacement connected to StackWise before it received power?

## Cisco documentation used

- *Managing Switch Stacks, Cisco Catalyst 9300 Series*
- *Understanding Cisco StackPower*
- *Configure and Troubleshoot StackPower on Catalyst 9300 Switches*
- *Cisco Catalyst 9300 Series Hardware Installation Guide*
- *Configuring High Speed Stacking, Cisco Catalyst 9300 Series*
- *Verify and Troubleshoot StackWise on Catalyst 9200 and 9300*
- *Upgrade Guide for Catalyst 9000 Switches*