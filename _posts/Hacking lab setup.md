---
title: Cyber Security Practice Lab
date: 2023-10-20 12:00:00 -0700
categories:
  - homelab
  - hardware
  - software
tags:
  - active-directory
  - blueteam
  - docker
  - hosting
  - proxmox
  - red-team
  - pfsense
  - windows
  - linux
---
# Intro
---
Welcome to my walkthrough on how to set up your very own Cyber Security lab. In this post I will be going over all the different systems to setup and how to network them together so that you can both break and deploy malware to your hearts content, down to defending and blocking all the bad things that can happen. This will just be a setup for the lab and a few examples on what you can do. I will not be going over any vulnerabilities or detection methods here. Now with that out of the way lets start it off.

# What you will need.
---
this lab is going to be pretty extensive so resource wise it will need a decently beefy system to host it on. 
System requirements:
	Minimum:
- 32GB RAM
- 1TB usable storage 
- quad core processor with hyperthreading (4core 8thread)
- usb ethernet adapter or PCIe ethernet NIC ( intel based )

Ideally you will want more storage and more cores for your processor but this will get you by.

Next we will need a few ISO images:
- Proxmox
- Windows Server 2019
- Windows 10
- Windows 11
- Security Onion
- PFSense
- Kali Linux
- Ubuntu Server
- Metasploitable VMs

you can ax the windows 11 or 10 box if you want but just know we will be setting up an active directory domain and connecting the client to the domain and knowing how to do that for both can help.

# Setting up your environment
---
To start it off you will need to install proxmox on your PC as the host OS. You can do so by following my guide [here](https://xxkaitonakamuraxx.github.io/posts/proxmox-install/). once that is done go ahead and install PFSense by following my guide [here](https://xxkaitonakamuraxx.github.io/posts/pfsense-install/). Now that you have your hypervisor and router/firewall installed and set up lets get down to configuring our Attack boxes.

## Active Directory
---

### Windows Server 2019
---

### Windows 10
---

### Windows 11
---








## Security Onion
---

### Linux Machines
---
almost their! Now for our linux environment we just need to set up our attack box (Kali) with our Ubuntu Server and Metasploitable VMs



