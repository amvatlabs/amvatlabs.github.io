---
layout: post
title: "Chapter 1 - Installing Proxmox"
date: 2025-10-19 12:50:00 +1100
authors: [avinash, angela]
categories: [homelab,virtualisation]
tags: [proxmox,installation]
description: Setting up Proxmox for our homelab with guidance from Jim’s Garage.
image: headers/01.png
media_subpath: /assets/img/posts/001-ProxmoxInstallation/
---

## What is Proxmox?

> "Proxmox Virtual Environment is a complete, open-source server management platform for enterprise virtualization. It tightly integrates the KVM hypervisor and Linux Containers (LXC), software-defined storage and networking functionality, on a single platform. With the integrated web-based user interface you can manage VMs and containers, high availability for clusters, or the integrated disaster recovery tools with ease." - [Proxmox VE](https://www.proxmox.com/en/products/proxmox-virtual-environment/overview)

## Installation

We have installed Proxmox in our homelab to create a reliable and flexible virtualization setup. However, we will not be covering the step-by-step installation process here because it is straightforward and widely documented online.

One of the best guides we found is **Jim’s Garage Homelab series** on YouTube. It provides clear instructions and tips for setting up Proxmox for home labs.

**Watch Jim’s Garage explain the step-by-step process for installing Proxmox in a homelab setup:**

{% include embed/youtube.html id='jNEjKdtMrZI' %}

*Video by [Jim’s Garage](https://www.youtube.com/watch?v=jNEjKdtMrZI) guided us through the installation process.*

With Proxmox installed, our homelab is ready for virtual machines, containers, and more advanced experiments, forming the backbone of our hands-on learning environment.

---
## Assigning a Static IP to the Proxmox Host

We need to assign a static IP to the Proxmox host to ensure reliable access to the Proxmox web interface.

Log in to your router and configure a DHCP reservation (binding) for the Proxmox host’s MAC address, assigning it a local static IP (in our case, 192.168.20.101).

> Note: Use any **available** local IP from your network range to avoid conflicts. 
{: .prompt-tip }

---
## Up Next

In the next chapter, we will be setting up Linux bridges for WAN and LAN (internal to Proxmox), allowing virtual machines and containers in Proxmox to communicate with each other and with devices on the home network.
