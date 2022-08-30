---
title: Windows Server 2019 Install
date: 2022-08-26 12:00:00 -0700
categories: [homelab, server]
tags: [windows,active-directory,dns] # TAG names should always be lowercase
---

# Active Directory Install
---
Today I will be going over my Windows Server installation. I will be using this as my domain controller by setting up Active Directory along with DNS for my local systems.


## Proxmox setup
---
1. To start it all off we will need the ISO for our install.
	1. You can download it [here](https://www.microsoft.com/en-us/evalcenter/download-windows-server-2022).
	2. This will only be able to be used for 180 days however I will outline a way to extend this timer if needed.
2. Once downloaded you can either go bare metal or virtual. for me I will be installing this on [Proxmox](https://fae-computing.tk/posts/proxmox-install/) so for this I will need to upload the ISO to the server over the web interface.

![](/assets/images/proxmox/os-upload.png)

3. Now that it is uploaded proceed to making the VM.

![](/assets/images/proxmox/win-general.png)
	1.  Name: windows-server-19

![](/assets/images/proxmox/win-os.png)
	2. ISO image: winserver2019.iso
	3. Type: Microsoft Windows
	4. Version: 10/2016/2019

![](/assets/images/proxmox/win-system.png)
	5. Qemu Agent: check mark

![](/assets/images/proxmox/win-disk.png)
	6. Disk Size: 50gib
	7. Bus/Device: SCSI
	8. SCSI controller: Virtio SCSI
	9. Cache: write back
	10. Discard: check mark
	11. Cores: 2 (this is all i needed)
	12. Memory: 4096 (again all i needed)
	13. Bridge: vmbr1 (this is my bridge for my [Pfsense]() install within Proxmox.)
4. Another thing we will need to do if installing on Proxmox is the virtio ISO drivers. this will improve compatibility and performance of windows when virtualized.
5. Download the virtio drivers [here](https://fedorapeople.org/groups/virt/virtio-win/direct-downloads/stable-virtio/virtio-win.iso).
	1. Upload the ISO to Proxmox.
	2. Now go to the hardware tab of the windows server VM and add a CD/DVD drive with virtio-win.iso selected.

![](/assets/images/proxmox/virtio-add-1.png)

![](/assets/images/proxmox/virtio-add-2.png)

6. Now you are ready to boot.


## Windows Install
---
1. Start following the installation wizard up to which OS you want to install. I chose the desktop version due to simplicity of using a GUI but feel free to install just the server core for a smaller attack surface over all.

![](/assets/images/win-serv/os-choice.png)

2. Get to custom or upgrade.

![](/assets/images/win-serv/custom-install.png)

3. Select custom and new.

![](/assets/images/win-serv/new.png)

4. Create
5. Your install should be running now. This will take a while.
6. Now to ensure all drivers are good head over to the device manager and click on anything with a driver issue.

![](/assets/images/win-serv/device-manager-1.png)

7. Click Update Driver.

![](/assets/images/win-serv/update-driver.png)

8. Select Browse my computer.

![](/assets/images/win-serv/browse-computer.png)

9. Browse to the virtio disk and check the include sub folders box and run.

![](/assets/images/win-serv/browse.png)

10. Once all are good you have a working base install. ( if you are in a VM like me now would be a good time to take a snapshot or backup of the VM in case we mess something up in the future ;)


## Install Active Directory & DNS
---
Now to install Active Directory and DNS on this server.

1. Head over to server manager located in the start menu if it did not auto start.
2. At the top right locate the manage button and select " Add roles and features".
3. Select next up till you get to server roles.

![](/assets/images/win-serv/install.png)

4. Select both "DNS" and "active directory domain services".
5. Continue and install.

![](/assets/images/win-serv/last-install.png)

6. now on the main page of server manager locate the flag next to the manage button.
7. you will see that the system needs to be promoted to a domain controller.
8. select the hyperlink in the notification.

![](/assets/images/win-serv/promote.png)

9. Select add a new forest.
10. Name it how you want.

![](/assets/images/win-serv/forrest.png)

11. Keep going till you get to the prereq checks (ignore the DNS error).
12. it should all pass and their will be a few errors however ultimately it should let you install.

![](/assets/images/win-serv/checks.png)

13. Once rebooted you now have a DNS and active directory domain controller! 
14. Next i will be going over how i configured them and connecting both a windows 10 and windows 11 systems to the domain in a later post.