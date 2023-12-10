---
title: PFSense Install v2
date: 2023-10-22 12:00:00 -0700
categories:
  - homelab
  - software
tags:
  - networking
  - pfsense
  - hosting
---

# PFSense Install
---
Hows it going again! its been over a year since my last pfsense setup and with me going through and revamping my lab to redo my Home SOC and testing grounds. i felt it was time to redo this little portion while i get the rest fixed up. without further adue lets get to it.
## Overview
---
## Installation
---
1. Like before you will need the ISO to get started. Go to [PFSense](https://www.pfsense.org/download/) and download it.

![](/assets/images/pfsense/pfsense-download.png)

- ensure to set the installer to **DVD Image (ISO) Installer**.

2. now upload the ISO to Proxmox

3. last time we went over to their guide to set this up but since i have a specific purpose with this lets walk through it

## For my specific setup
---
![](/assets/images/proxmox/pve-network-bridge.png)

- i had to ensure that the bridge ports section states my ethernet adapters name located under the network tab for proxmox

![](/assets/images/proxmox/pve-network-tab.png)

4. now unlike the instructions provided by PFSense for installation i used vmbr0 as the WAN and vmbr1 for the LAN. the reason why is so that i have a seperate virtualized router that is still able to provide outside internet to the VMs behind it. this way i can still manage it over my ISP provided router along with internaly via the VMs or hardlined to the VM.



## Physical Access
---
to connect my real devices to the network I used vmbr1 to connect to an 8 port managed switch. this will be used for future expansion when needed.

## Conclusion
---
thats pretty much it! while using the same port proxmox is using for internet is not best practice in a pinch its working for my situation until i can build a dedicated PFSense machine.

Thank you for reading!
