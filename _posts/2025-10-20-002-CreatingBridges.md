---
layout: post
title: "Chapter 2 - Creating Bridges"
date: 2025-10-20 12:00:00 +1100
authors: [avinash, angela]
categories: [homelab,virtualisation]
tags: [proxmox,configuration]
description: Creating LAN and WAN bridges in Proxmox to connect virtual machines.
image: headers/01.jpg
media_subpath: /assets/img/posts/002-CreatingBridges/
---

## Introduction
In this chapter, we will create the network bridges in Proxmox that will be used for the internal and external networks. These bridges connect the virtual machines to the physical network interfaces and allow communication between the internal LAN and the WAN.

> **Note:** From here on, **external network or WAN** refers to our home network (192.168.20.0/24), while **internal network** refers to the network within Proxmox. Do not confuse the external network with the internet. 
{: .prompt-warning }

---
## Steps to Create Bridges
1. Log in to the Proxmox web interface.
2. Navigate to **Datacenter → Node → System → Network**.
![001](001.png)
3. Click **Create → Linux Bridge** to add a new bridge.
4. For the WAN bridge:
   - Set the bridge name to `vmbr0`.
   - Set the IPv4 address to `192.168.20.101/24` and the gateway to `192.168.20.1`. The IP `192.168.20.101` is the static IP configured on the router for the Proxmox host and is used here as the IPv4 address for the WAN bridge.
   - Leave the remaining settings as default.
   - Click **Create**.
![002](002.png)
5. For the LAN bridge:
   - Click **Create → Linux Bridge** again.
   - Set the bridge name to `vmbr1.
   - Do not assign it to a physical interface; this will be used as an internal LAN.
   - Leave other settings as default.
   - Click **Create**.
![003](003.png)
6.  Once both bridges are created, verify that they appear in the network list.
![](004.png)
7. Apply the configuration changes and reboot the node.

---
## Conclusion
With the bridges in place, virtual machines can now be connected to the appropriate network segments (WAN and LAN), enabling proper network communication for subsequent VM deployments.

---
## Up Next
In the next chapter, we will deploy the OPNsense firewall in Proxmox. With the network bridges already in place, OPNsense can be connected to both the WAN and LAN, allowing it to function as the central firewall and router for the internal network.