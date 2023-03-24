---
title: 9.1 Plex LXC
parent: 9. Plex Specifics
has_children: false
nav_order: 1
---

### 9.1 Plex LXC

Configure a new Proxmox Plex LXC using the script from tteck's helpers:
[https://tteck.github.io/Proxmox/](https://tteck.github.io/Proxmox/)

Use the following parameters:
* Privileged container
* Ubuntu 20.04 template
* 40GB storage
* 6 cores
* 8192MB memory
* Static IP address (I used 172.16.10.222/24)
* DNS domain: local
* DNS address: 172.16.10.2 (which is pihole)
* Gateway: 172.16.10.1
* ipv6: yes
* root SSH: no
* Verbose: no

Allow it to start up. Then we can mostly follow this guide, but most of the work is done for us:
[https://www.razib.io/dev/proxmox-plex-setup-on-ubuntu-lxc-with-intel-11th-generation-cpu-and-quick-sync-transcoding/](https://www.razib.io/dev/proxmox-plex-setup-on-ubuntu-lxc-with-intel-11th-generation-cpu-and-quick-sync-transcoding/)

The helper script should have already set up hardware transcoding. Verify if by doing:
`ls -l /dev/dri`

Note that the i915 device should not be blacklisted in proxmox, but make sure you blacklist the frame buffer so that proxmox doesn't grab the iGPU. At the time of writing my `nano /etc/kernel/cmdline` looked like this:
`root=ZFS=rpool/ROOT/pve-1 boot=zfs quiet intel_iommu=on iommu=pt pcie_acs_override=downstream,multifunction initcall_blacklist=sysfb_init video=simplefb:off video=vesafb:off video=efifb:off video=vesa:off disable_vga=1 vfio_iommu_type1.allow_unsafe_interrupts=1 kvm.ignore_msrs=1 modprobe.blacklist=bnx2x,ahci,igb,r8169`

From here, you can then skip ahead to the part where you install curl, gnupg, and most importantly the intel drivers.

Obviously you can skip installing plexmediaserver, because it is already installed.

Reboot the LXC.


