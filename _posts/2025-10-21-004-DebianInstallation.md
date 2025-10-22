---
layout: post
title: "Chapter 4 - Installing Debian in Proxmox"
date: 2025-10-21 12:50:00 +1100
authors: [avinash, angela]
categories: [homelab,endpoint]
tags: [debian,installation]
description: Setting up a Debian VM as the internal admin endpoint.
image: headers/01.jpg
media_subpath: /assets/img/posts/004-DebianInstallation/
---

## Introduction
In this chapter, we’ll set up a Debian virtual machine that acts as the heart of our internal network management. This VM will serve as the internal admin endpoint in Proxmox, providing a secure and dedicated space to access and manage OPNsense and other internal services through the LAN.

---
## Downloading the Debian ISO
Download the latest Debian ISO from the official [Debian](https://www.debian.org/download) website.

---
## Creating the Debian VM
1.  Log in to the Proxmox web interface.
2. Go to the Proxmox storage where ISO images are kept and upload the downloaded Debian ISO.
3. Click “Create VM" and give it a name (e.g., Debian-Internal-Admin).
![](001.png)
4. Under the “OS” tab, select “Use CD/DVD disc image file (ISO)” and choose the uploaded Debian ISO.
![](002.png)
5. Choose the default system settings.
![](003.png)
6. Add a disk for installation and set the size to 20 GB.
![](004.png)
7. Assign 2 CPU cores.
![](005.png)
8. Assign memory of 4096 MB (4 GB).
![](006.png)
9. Use the default network interface. This will be changed later.
![](007.png)
10. Review and save the configurations.
![](008.png)
11. Next, we need to change the existing network interface and add a new one. Under “Hardware,” double-click the network interface and change it from `vmbr0` to `vmbr1`. This is the network device that will be connected to the LAN of OPNsense.
![](009.png)
12. This is the final network configuration.
![](010.png)
13. Launch the VM and complete the initial setup of Debian, including creating the user and setting up credentials.
14. To test connectivity to OPNsense, open a web browser within the Debian VM and navigate to `192.168.1.1`, which is the OPNsense interface address obtained after rebooting. Below is the screenshot for reference:
![](011.png)

---
## Testing Browser Connectivity 
1. Open a web browser and navigate to **192.168.1.1**. The screenshot below shows that the OPNsense web interface can be accessed from the Debian internal admin VM.
![](012.png)

---
## Conclusion
In this chapter, we successfully deployed the Debian internal admin VM and verified that it can access the OPNsense web interface over the internal LAN. This setup establishes a secure endpoint for managing OPNsense and other services within the internal network.

---
## Up Next
In the next chapter, we will complete the initial configuration of OPNsense to ensure the network setup is properly established. 