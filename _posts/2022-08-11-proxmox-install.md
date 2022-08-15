---
title: Proxmox Install
date: 2022-08-11 12:00:00 -0700
categories: [homelab,software]
tags: [proxmox,hosting,servers] # TAG names should always be lowercase
---

# Proxmox Install
---


## Reason
---
As discussed in my [Homelab post](https://fae-computing.tk/posts/homelab/) as many who want to start off with homelab, money is an issue. therefore virtualization is our best friend. After trying out different hypervisors such as hyper-v and vmware's ESXI. I settled on Proxmox due to its prevelence in the homelab space and its ease of use. main reason is that it is free lol.



## Overview
---
for this instillation i will be using it to virtualize my entire homelab and have the option to expand with other nodes if needed. I wanted to be able to simulate enterprise networks with no worries about breaking systems due to snapshots and easily expand if needed.



## Instilation
---
1. download the ISO from [Proxmox website](https://www.proxmox.com/en/downloads/category/iso-images-pve) 

![](/assets/images/proxmox/download-pve.png)

2. select either the direct download or Torrent.
3. now using the software of your choosing burn the ISO to a thumb drive. personaly i like balena etcher or simply dd or cat in linux when internet is not available
4. plug in your ethernet cable at this point. it will save time with some of the network setup later.
5. now after booting into Proxmox we begin by seting up which drive we want our install to go to

![](/assets/images/proxmox/choose-disk-pve.png)

6. personaly i used an nvme ssd for my inernal boot disk. keep in mind that the drive you use for the install will also house the backups and Disk images in the future.
7. next we will select the timezone and location for the server. this will keep track of the system time.

![](/assets/images/proxmox/location-pve.png)

8. after this we will setup the administrator account credentials. you will setup a root password along with providing a valid email for support services if you want and for alerts from the server.

![](/assets/images/proxmox/password-pve.png)

9. now we are at one of the most important steps, networking. now if you have pluged in an ethernet cable before booting it should populate automaticaly with a gateway address

![](/assets/images/proxmox/network-pve.png)

10. now chose the management interface you would like to use. i.e. the ethernet port that is giving internet to your system
11. for the hostname chose what you want. for my homelab i have no need for a public domain name here so i setup a private one.
12. for the ip address it should auto populate if connected to a DHCP server, or simply your router. if not, manualy imput an ip within the address range of your DHCP server.
13. for the gateway, again it should auto populate. if not then see your router/DHCP to what its ip is.
14. for DNS you can point this to an internal dns if you have it previously setup or you can use other public DNS servers. personaly i like cloudflare's DNS of : **1.1.1.1** 
15. now procced through the instillation and reboot.



## Conclusion
---
once rebooted procede to the web interface on ( https://IP-of-machine:8006 )

Congradulations! you know have a full up hypervisor setup for your cheap homelab needs!