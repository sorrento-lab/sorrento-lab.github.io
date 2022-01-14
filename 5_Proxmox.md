---
title: "5. Proxmox Configuration"
has_children: true
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

To a quick update of proxmox:  
`apt-get update`  
`apt-get dist-upgrade`  

In system -> network, add a OVS bridge vmbr1 and click autostart = on. No need to fill out other parameters.
The management interface eno1 can remain bonded via linux bridge on vmbr0. eno1 will be hard plugged to the switch rather than virtually administered.
