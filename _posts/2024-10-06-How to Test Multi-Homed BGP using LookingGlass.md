---
title: How to view BGP PathPrefix in LookingGlass
description: Mirror Mirror on the Wall!
date: 2024-10-06 12:00:00:01 +/-0000
categories: [NetworkEngineer, Routing]
tags: [bgp,failover,lookingglass,asn,cisco]     # TAG names should always be lowercase
toc: true
comments: active
ping: true
image:
  path: '/assets/img/headers/proxmox.webp'
pin: false
published: false
---
# Scenario
Depending on who you work for and how large your network is, its possible you might be multi-homing your prefix's out towards the internet.  Multi-homing is when you want to advertise a public ip prefix out two different ISP's. You will obviously need to set a weighted value to make sure preference is seen over the primary ISP circuit when it is up and passin traffic.  

The common use case scenario, is that you have a location where you have to internet service Providers.  You want everybody to reach your site using your primary circuit.  In the event your connectivity to the internet goes down with that circuit, you need to make that similar prefix available out your secondary/backup circuit hopefully provided by a secondary ISP using a diverse path.   

# Tools
I always think its a good idea to test your Disaster Recovey(DR) scenarios on a routine basis.  This not only ensures your DR works, but at the same time, it gives the team the practice, experience, and confidence to complete in the event an actual issue occurs.  

When confirming BGP I use a few tools. 

- LookingGlass (https://lg.he.net/)
- Cacti
- Ping and Traceroute from an internet Site (ping.pe)
- Pinginfoview (from my pc reaching few IP's on our network during failover)

# What is BGP
To understand BGP, you have to first understand how a packet is routed.  If a router receives a packet on an interface, it looks at its routing table, and determines which interface to send the packet.  This is a very basis single router to single router type communication. 

With BGP, you have AS Numbers assigned.  An AS is assigned to a collection of routers. 

{% include embed/youtube.html id='RBjeB9M1dGQ?si=bdusDX8I5fyPTcgj' %}


# Conclusion 



# Resources
1. I watched this video
https://www.youtube.com/watch?v=IQA7aTezrVE

{% include embed/youtube.html id='IQA7aTezrVE' %}

2. Helped me understand the ID and UUID section
https://dannyda.com/2020/09/24/how-to-add-attach-new-disk-to-existing-zfs-pool-on-proxmox-ve-pve-how-to-remove-detach-disk-from-zfs-pool-on-proxmox-ve/#google_vignette

3. Future resource on how to replace Boot Drive
https://theorangeone.net/posts/proxmox-boot-drive-replacement/
