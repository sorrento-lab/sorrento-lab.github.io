---
title: 6.1 VM configuration
parent: 6. OPNsense Configuration
has_children: false
nav_order: 1
---

OPNsense will be run as a guest within Proxmox. The disadvantage of this is that if Proxmox is down, then OPNsense is down. Currently no Proxmox or OPNsense related crashes have been experienced, if a crash is observed then a CARP setup will be investigated. The advantage is lower power consumption (single, rather than multiple devices) and the ability to take snapshots between updates.

Set up a new VM with the following parameters:
* VM ID `100`
* Name `OPNsense`
* Memory `8 GB`
* Processors `4 vCPUs`, type kvm64 and add flag `aes-ni` to get passthrough of this function of the CPU
* BIOS and machine leave as default SeaBIOS and i440fx
* Add local iso OPNsense installer (upload the OPNsense iso beforehand via local -> ISO images in the GUI)
* Add a 40G vdisk, enable SSD emulation and IO disk
* Add vmbr1 as a network device, use the virtIO driver and set multiqueues to 8 **(to be investigated - does multiqueues=4 result in better performance? Is multiqueues supported for Broadcom cards?)**
* Add the SFP+ card as a PCIe device. Pass through the entire card, using "all functions = on" and "ROM bar = on"
* You will also need to temporarily pass through a second NIC (NOT the management interface of proxmox). I used the spare 2.5G realtek on my board. This is to create the LAN bridge easily.

Once created, in VM 100 -> Options
* Start at boot = yes
* Startup order = 1, startup delay 120 (to allow OPNsense services to start running before other VMs boot)
* Use tablet for pointer = no
* QEMU guest agent = enabled

References:  
[https://homenetworkguy.com/how-to/run-opnsense-in-proxmox-vm/](https://homenetworkguy.com/how-to/run-opnsense-in-proxmox-vm/)  
[https://pfstore.com.au/blogs/guides/run-opnsense-in-proxmox](https://pfstore.com.au/blogs/guides/run-opnsense-in-proxmox)  
