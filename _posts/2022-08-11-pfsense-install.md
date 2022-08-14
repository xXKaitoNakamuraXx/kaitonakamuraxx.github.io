---
title: PFSense Install
date: 2022-08-11 12:00:00 -0700
categories: [homelab,software]
tags: [networking,pfsense,hosting] # TAG names should always be lowercase
---

# PFSense Install
---

## Reason
---
I wanted to have my home network segregated so that I wont have to worry about any other devices interfeering or my home network going down when I am tinkering ( those of you with wives will understand... )



## Overview
---
here i will be covering the install of PFSense on Proxmox and not the installation of proxmox its self. please see my [Proxmox Install](https://fae-computing.tk/posts/proxmox-install/) before i begin i will say that my situation is less than ideal but if their is a will their is a way! 

I deciede due to limited funds and spare hardware I would be virtualizing this in proxmox and using it as the main router for my homelab both VMs and Hardware. Doing this was not difficult at all. best part if a config I make bricks it i have snapshots for a simple and easy restore to my last good configs. keeping my network uptime as high as possible. enough of the semantics lets get to the bread and butter!



## Installation
---
1. First things first you will need the ISO. Go to [PFSense](https://www.pfsense.org/download/) and download it.

![[/assets/images/pfsense/pfsense-download.png]]

- ensure to set the installer to **DVD Image (ISO) Installer**.

2. now upload the ISO to Proxmox

3. Follow the guide given by PFSense for [instilation with Proxmox](https://docs.netgate.com/pfsense/en/latest/recipes/virtualize-proxmox-ve.html) for a working standard.
	- given my special situation with how i need to setup my network highlighted [here](https://fae-computing.tk/posts/homelab/#Networking) I needed to make a few adjustments.



## For my specific setup
---
1. instead of using an ethernet NIC to add ports for the device. All i had was a spare gigabit usb ethernet adapter.
2. setup is the same as the given instructions however i needed to use the ethernet port on the MotherBoard for both proxmox and the wan of the PFSense VM.
3. so when going through the setup instead of setting up both vmbr1 & 2 i only create vmbr1 using the ethernet dongle as the port

![[/assets/images/proxmox/pve-network-bridge.png]]

- i had to ensure that the bridge ports section states my ethernet adapters name located under the network tab for proxmox

![[/assets/images/proxmox/pve-network-tab.png]]

4. now unlike the instructions provided by PFSense for installation i used vmbr0 as the WAN and vmbr1 for the LAN. the reason why is so that i have a seperate virtualized router that is still able to provide outside internet to the VMs behind it. this way i can still manage it over my ISP provided router along with internaly via the VMs or hardlined to the VM.



## Physical Access
---
to connect my real devices to the network I used vmbr1 to connect to an 8 port managed switch. this will be used for future expansion when needed.

## Conclusion
---
thats pretty much it! while using the same port proxmox is using for internet is not best practice in a pinch its working for my situation until i can build a dedicated PFSense machine.

Thank you for reading!