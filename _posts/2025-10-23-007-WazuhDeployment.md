---
layout: post
title: "Chapter 7 - Wazuh Deployment"
date: 2025-10-23 12:00:00 +1100
authors: [avinash, angela]
categories: [homelab,siem]
tags: [wazuh,deployment]
description: Deploying Wazuh to monitor and collect internal security events.
image: headers/01.jpg
media_subpath: /assets/img/posts/007-WazuhDeployment/
---

## Introduction
This series covers deploying the free and open-source Wazuh server inside Proxmox. The Wazuh server will monitor security events and collect logs from endpoints within the internal network.

---
## Prerequisites
The Wazuh central components require a 64-bit Linux system to run. For this deployment, we will be installing **Ubuntu 24.04 Server**. Other supported operating systems are also available on the [Wazuh documentation website](https://documentation.wazuh.com/current/quickstart.html).

Download the Ubuntu Server ISO image from [Ubuntu](https://ubuntu.com/download/server).

We are using the Ubuntu Server edition instead of the Desktop version because it is lightweight, uses less disk space, and avoids unnecessary graphical packages, making it more efficient for server deployments in Proxmox.

---
## Creating the Wazuh Server VM
1. Log in to the Proxmox web interface.
2. Go to the Proxmox storage where ISO images are kept and upload the downloaded Ubuntu ISO.
3. Click “Create VM" and give it a name (ie. Wazuh-Server).
![](001.png)
4. Under the “OS” tab, select “Use CD/DVD disc image file (ISO)” and choose the uploaded Ubuntu Server ISO.
![](002.png)
5. Choose the default system settings.
![](003.png)
6. Add a disk for installation and set the size to 50 GB.
![](004.png)
7. Assign 4 CPU cores and set the CPU type to Host, as it exposes all real CPU features to the VM, ensuring maximum performance for Wazuh.
![](005.png)
8. Assign memory of 8192 MB (8 GB).
![](006.png)
9. Add a network interface using `vmbr1`, which connects to the internal LAN.
![](007.png)
10. Review and save the configurations.
![](008.png)

---
## Assigning a Static IP and VLAN Tag to the Wazuh Server
After creating the Wazuh server VM, we need to assign it a static IP address on the internal LAN and tag it with the appropriate VLAN for security tools and services.
1. In OPNsense, navigate to **Services → ISC DHCPv4 → SecTools**. Enter the MAC address of the Wazuh server VM and assign a static IP `192.168.2.2`. Save the configuration.
![](009.png)
![](010.png)
2. The image below shows the DHCP static mappings for this interface.
![](011.png)
3. Return to Proxmox, open the VM’s **Hardware** settings, select the network interface connected to `vmbr1` (LAN), and set the **VLAN tag** to `2`.
![](012.png)
4. Reboot the Wazuh server VM. After it restarts, open the console and run `ip a` to verify that the static IP has been correctly assigned.
![](013.png)
5. Open the Wazuh server console and run the installation command using `curl` as provided in the [Wazuh Quick Start guide](https://documentation.wazuh.com/current/quickstart.html).
![](014.png)
6. Once the installation is complete, note the details provided for accessing the Wazuh web interface.
![](015.png)

---
## Conclusion
In this chapter, we successfully deployed the Wazuh server on Ubuntu Server within Proxmox, assigned it a static IP on the internal LAN, and configured the appropriate VLAN for security tools.

---
## Up Next
In the next chapter, we will enable access to the Wazuh web interface (GUI) from the WAN, i.e. our home network.