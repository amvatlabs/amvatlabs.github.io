---
layout: post
title: "Chapter 3 - Deploying Opnsense Firewall"
date: 2025-10-21 12:00:00 +1100
authors: [avinash, angela]
categories: [homelab,firewall]
tags: [opnsense,installation]
description: Deploying OPNsense as a virtual firewall to secure lab network traffic.
image: headers/01.jpg
media_subpath: /assets/img/posts/003-DeployingOpnsense/
---

## Introduction
In this chapter, we will guide you through deploying OPNsense as a virtual firewall in Proxmox. OPNsense will serve as the central point of control for our lab network, managing traffic between the WAN and LAN while providing essential security features like NAT and firewall rules. 

---
## Prerequisites
- Proxmox is installed and accessible.
- Network bridges (`vmbr0` for WAN and `vmbr1` for LAN) already created.
![](001.png)

---
## Downloading and verifying the OPNsense ISO 
1. Go to the official [OPNsense](https://opnsense.org/download/) website.
2. Choose the correct architecture (usually AMD64) and installer type (DVD ISO).
![](002.png)
3. Save the ISO file to your local machine. 
4. Before deploying OPNsense in Proxmox, it’s important to ensure the ISO file is authentic and hasn’t been tampered with.
5. Verify the ISO using the SHA256 checksum:
![](003.png)
6. Compare the output with the SHA256 checksum provided on the OPNsense download page. They must match exactly.
![](004.png)
7. Once verified, the ISO is safe to upload to Proxmox for VM creation.

---
## Creating the OPNsense VM
1. Log in to the Proxmox web interface.
2. Go to the Proxmox storage where ISO images are kept and upload the verified OPNsense ISO.
![](005.png)
3. Click “Create VM” and give it a name (e.g., OPNsense).
![](006.png)
4. Under the “OS” tab, select “Use CD/DVD disc image file (ISO)” and choose the uploaded OPNsense ISO.
![](007.png)
5. Choose the default system settings.
![](008.png)
6. Add a disk for installation and set the size to 32 GB.
![](009.png)
7. Assign 4 CPU cores.
![](010.png)
8. Assign memory of 4096 MB (4 GB).
![](011.png)
9. Use the default network interface. This will be changed later.
![](012.png)
10. Review and save the configurations.
![](013.png)
11. Next, we need to change the existing network interface and add a new one. Under “Hardware,” double-click the network interface and change it from `vmbr0` to `vmbr1`. This is the network device that will be connected to the LAN configured in Proxmox. (Refer to the previous chapter for more details.)
![](014.png)
12. Add a new network device by clicking “Add” → “Network Device” and choose the interface as `vmbr0`. This network interface will serve as the WAN of OPNsense and is attached to the NIC of Proxmox.
![](015.png)
13. This is the final network configuration.
![](016.png)
14. Before booting up the OPNsense virtual machine, use the MAC address of the WAN interface (`vmbr0`) to configure a DHCP reservation on your home router. Assign it the static IP address **192.168.20.102**. This ensures that the OPNsense machine retains the same IP address even after restarts, allowing consistent access to the web interface in the future.

---
## Initial OPNsense Setup
1. Once the OPNsense VM is launched, the FreeBSD installer screen will appear. Choose “Continue with default keymap.”
![](017.png)
2. In the OPNsense installer, choose “Install (UFS).”
![](018.png)
3. Under UFS configuration, select the disk `da0 <QEMU QEMU Harddisk 25>` (32 GB).
![](019.png)
4. Choose “Yes” when prompted to continue with a recommended swap partition of 8 GB.
![](020.png)
5. Choose “Yes” to destroy the current contents of the `da0` disk.
![](021.png)
6. Select the option to change the root password to the desired one.
![](022.png)
7. Complete the installation process.
![](023.png)
8. Reboot the system once installation is finished.
![](024.png)
9.  Below is the output after rebooting the system.
![](025.png)
The LAN interface is set to **192.168.1.1/24** by default. The WAN interface receives the IP address **192.168.20.102/24** from the DHCP server at the home router. 

---
## Up Next
Next, we’ll deploy the Debian Internal Admin VM and verify access to the OPNsense web interface from the internal network.
