---
layout: post
title: "Chapter 0 - Meet Our Lab Gear"
date: 2025-10-19 12:00:00 +1100
authors: [avinash, angela]
categories: [homelab,hardware]
tags: [server,hp]
description: A quick look at the hardware powering our homelab.
image: headers/01.jpg
media_subpath: /assets/img/posts/000-HardwareOverview
---

## HP ProDesk 400 G6 SFF Desktop

For our homelab setup, we are using a **refurbished HP ProDesk 400 G6 Small Form Factor (SFF) desktop** as our workstation. It is powered by an **Intel Core i5-9600 processor** running at **4.6GHz**, providing solid performance for virtualization, server tasks, and lab experiments.

With **32GB of RAM**, the system can comfortably handle multiple virtual machines, containers, and testing environments simultaneously. Storage is provided by a **1TB SSD**, ensuring fast boot times, quick application launches, and ample space for our lab data. It comes pre-installed with **Windows 11**, but we **will be replacing it with Proxmox** to better suit our virtualization and lab management needs. The desktop cost us **777.99 AUD**.

![HP ProDesk 400 G6 SFF Desktop](001.jpg)
_HP ProDesk 400 G6 SFF Desktop_

---
## Networking

As the desktop lacks a built-in wireless adapter, we decided to use a **wired connection** for reliable performance. An **Alogic Cat6 Ethernet cable** connects the desktop directly to our home router, costing **25 AUD**.

![Alogic Cat6 Ethernet cable](002.jpg)
_Alogic Cat6 Ethernet cable_

---
## Monitor and Stand

To facilitate the initial setup of the desktop, we have purchased an **MSI Pro MP245V 23.8" Full HD 100Hz Business Monitor**, costing **99 AUD**. To keep the desktop organized and mobile, we also have a **Hovadova adjustable PC stand with caster wheels**, priced at **40 AUD**. 

![MSI Pro MP245V 23.8" Full HD 100Hz Business Monitor](003.jpg)
_MSI Pro MP245V 23.8" Full HD 100Hz Business Monitor_

![Hovadova adjustable PC stand with caster wheels](004.jpg)
_Hovadova adjustable PC stand with caster wheels_

---
## Up Next

In the next chapter, we will be installing **Proxmox** on our HP ProDesk 400 G6 desktop. This will allow us to set up a virtualized environment for our homelab, enabling multiple virtual machines and containers to run efficiently.