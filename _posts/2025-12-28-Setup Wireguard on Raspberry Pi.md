---
title: Configure a Raspberry Pi to connect to your home network.
description: Basic steps of how to configure a Raspberry Pi 5 to connect to your home network with Wireguard VPN services and Unifi.
date: 2025-12-28 12:00:00:01 +/-0000
categories: [Homelab, RaspberryPi]
tags: [wireguard,unifi,vpn]
mermaid: true
toc: true
comments: true
ping: true
#image:
#  path: '/assets/img/headers/gitaddremove.webp'
pin: false
published: true
# permalink: /series/
seo:
  title: "Configure a Rapsberry Pi to connect to your home network"
  description: "Configure your Raspberry Pi to connect to your Unifi home network using Wireguard VPN"
  # canonical: "https://wangineer.com/series/"
  keywords: "wireguard, vpn, unifi"
---
## Scenario
I'm on vacation and want to continue to have access to services at my home.  I have really been enjoying the use of Wireguard as it has posed to be very simple.  

During this tutorial, I am not going to provide the steps on how to setup the server.  Maybe at a later date. 


## The Steps

Step 1. Update your Pi
```
sudo apt update && sudo apt full-upgrade -y
```

Step 2. Install Wireguard tools
```
sudo apt install wireguard-tools -y
```

Step 3. Log into your Unifi controller and provision a client. Download the file to the desktop.

Note: If you are doing this via CLI you can create the file by using the "touch" and then copy and paste the configuration file details into the document.

Example file:
All details will be already filled out based on your configuration.
```
[Interface]
PrivateKey = <Your_Client_Private_Key>
Address = 10.0.0.2/32
DNS = 1.1.1.1

[Peer]
PublicKey = <Your_Server_Public_Key>
Endpoint = <Server_IP_or_Domain>:51820
AllowedIPs = 0.0.0.0/0
```
The below commands create a file and allows you to get into the file using nano.  Feel free to use vi or any other text editor. 
```
touch your_config.conf
nano your_config.conf
``` 

Step 4: Move the file from your desktop to the wireguard folder and rename it "wg0.conf"



Note: I mentioned the desktop above, but the key take away is that the command below just makes it so that you have to move it from the current working directory that you are in.
```
sudo mv your_config.conf /etc/wireguard/wg0.conf
```


Step 5: Change permission of the file. 
```
sudo chmod 600 /etc/wireguard/wg0.conf
```

Step 6: Because I am running a recent OS of Raspberry PI there seems to be a conflict with "resolveconf" and "network manager" 

Initially the steps state to install resolvconf, but this ends up breaking your DNS and ability to resolve anything and get to the intenret. Since Raspberry PI OS comes with the Network Manager, nothing else needs to be installed.  YOu just need to import the Wireguard Configuration file into the tool. 
```
sudo nmcli connection import type wireguard file /etc/wireguard/wg0.conf
```

Step 7: Start the VPN
```
sudo nmcli connection up wg0
```

Step 8: Check the status of VPN
```
nmcli connection show --active
sudo wg show 
sudo wg show wg0
nmcli connection show wg0

ip link show wg0   - TElls you if vpn interface is up or down
```

Step 9: Have the vpn start on boot up
```
sudo nmcli connection modify wg0 connection.autoconnect yes
```


## If you need to install resolve conf there are different steps
Step 1: Install resolvconf the same time you installed wireguard in the above steps
```
sudo apt install resolvconf
```
Step 2: You don't need to import the file into the nmcli like you did above 

Step 3: Bring VPN up
```
sudo wg-quick up wg0
```

Step 4: Check the status
```
sudo wg show
```

Step 5: Bring Vpn Down
```
sudo wg-quick down wg0
```

Step 6: Have the VPN startup ever time it boots up
```
sudo systemctl enable wg-quick@wg0
```




Step 3: Get configuration from Unifi Controller config file
download and touch the file on the desktop. 


Step 5: Set permissions


Step 5: MV the file into network manager folder
sudo nmcli connection import type wireguard file /etc/wireguard/wg0.conf


Step X: Start the VPN
sudo nmcli connection up wg0


Step X: Have the vpn start on boot up
sudo nmcli connection modify wg0 connection.autoconnect yes


If you are using NetworkManager (standard on Raspberry Pi OS Bookworm/2025), wg-quick still expects the resolvconf command to be present whenever a DNS = line is in your configuration. 
You have two primary ways to resolve this while keeping NetworkManager as your primary DNS tool:

1. The "Native" Way: Import into NetworkManager
Instead of using wg-quick, you can import your configuration directly into NetworkManager. This is the cleanest method because it uses NetworkManager's native WireGuard support and avoids the resolvconf error entirely. 










old

Step 6: Start the connection
sudo wg-quick up wg0


Step 7: Check the Status
sudo wg show wg0



Step 8: Enable at start on boot
sudo systemctl enable wg-quick@wg0

Step 9: Validate your public ip is now of the server connection side and not where you are at

Step 10: Disconnect the Wireguard VPN
sudo wg-quick down wg0

















### Fast Forward Merge - Used when no conflicts are present
> NOTE: When completing a Fast Forward merge, you want to be in the target branch. So in this case, we want to merge the changes we made in bumblebee into "main".  So to do this, we need to be in the "main" branch
{: .prompt-danger }


```
git checkout main  <- This ensures you are in the correct target branch
git merge bumblebee
git log --oneline
```
  The above moves the Main Branch from the previous (ancestor of bumblebee) so it can just be moved up
  This works when no changes to main and only changes were made to bumblebee.  
  This basically just changes the pointer fromthe old hash to the new and accepts all the previous commits 


### Deleting a branch
Lets say you messed up completely and want to start again, or the team says we no longer need that feature.  So its timeto delete all work. 
```
git branch -d bumblebee   < This deletes the "bumblebee" branch completely

if you made changes and states the branch is not fully merged, you may need to replace the -d with -D
git branch -D bumblebee
```


## Advanced Branch creation feature
Lets say you want to skip creating a branch with one line of code, and moving it into the next line of code
```
git checkout -b bumblebee2 main
  - -b means to create bumblebee2 and base it upon the "main" branch.  
git branch  < Check to see that you have moved into that branch  
```







## Call to action
ASK CHAT GPT What a call to action is since you already did the lab work above. 


## Quick Commands
```
--- Shows history of commits
git log --oneline
--- Verify the main commit line that should match the "HEAD" "git log --oneline"
cat .git/refs/heads/main
--- Create a new Branch
git branch bumblebee
--- List the branches and verify which branch you are in
git branch
ls .git/refs/heads/
--- Move into that newly created branch
git checkout bumblebee
or
git switch bumblebee
--- Fast Forward Merge make sure you are in the target branch that is staying
git checkout main
git merge bumblebee
--- Advanced create branch and directly move into that branch
git checkout -b bumblebee main
--- To discard a merge if there are conflicts you don't want to deal with at the moment
git merge --abort
```