---
title: "5. Proxmox Configuration"
has_children: false
nav_order: 5
---

## 5. Proxmox configuration

The proxmox guided installer is fairly self explanatory. The following settings are used:
* Mirrored ZFS Raid 1. No need to partition discs here, just use the whole disk(s).
* Management interface `eno1` - I picked an intel 1G interface here
* Hostname: `proxmox.local`
* Static IP of `172.16.10.5` or if setting up from livebox initially, use 192.168.1.5
* Subnet of `255.255.255.0` (/24)
* Gateway of `172.16.10.1` or if setting up from livebox initially, use 192.168.1.1
* DNS `1.1.1.1`

The IP addresses can be changed later once OPNsense is set up.

To remove the subscription nag:
[https://github.com/Jamesits/pve-fake-subscription](https://github.com/Jamesits/pve-fake-subscription)
Ensure that this is removed prior to any distr-upgrade of proxmox, and check if it is still valid for the PVE version you upgrade to before re-installing.

To do an upgrade of proxmox (PVE and kernels):  
`apt-get update`  
`apt-get dist-upgrade`  

To just update packages:
`apt-get update`  
`apt-get upgrade`  

To recreate self-signed certificates:
`pvecm updatecerts --force`
Reference: [https://forum.proxmox.com/threads/rebuild-ssl-certificate-for-updated-domain-name.25057/](https://forum.proxmox.com/threads/rebuild-ssl-certificate-for-updated-domain-name.25057/)

eno1 is bridged to vmbr0. It is possible to add more devices to this virtual birdge for a "all in one box" setup, but I found the virtual bridge does not like more than 3 physical devices on it. Instead, bridge everything inside opnsense. The disadvantage of this is that if you log into proxmox usually via wifi, you will not be able to do so until opnsense and pihole are up. It is still possible to physically access proxmox interface for debugging via eno1.
