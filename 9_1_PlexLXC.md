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

Now follow steps 1,3 and 6 of this guide:
[https://forums.serverbuilds.net/t/guide-hardware-transcoding-the-jdm-way-quicksync-and-nvenc/1408/5](https://forums.serverbuilds.net/t/guide-hardware-transcoding-the-jdm-way-quicksync-and-nvenc/1408/5)

Go to `172.16.10.222/manage` and import your libraries. While that is importing, create a new port forward rule in OPNsense to `172.16.10.222/32400` for remote access.

Make sure you re-link your tautulli stats collector.

To monitor the intel iGPU, we will re-use a small script developed here:
[https://github.com/brianmiller/docker-intel-gpu-telegraf](https://github.com/brianmiller/docker-intel-gpu-telegraf)
but with a few changes. In the LXC console, install intel gpu tools:

`apt install intel-gpu-tools`

and then install telegraf:
`# influxdata-archive_compat.key GPG Fingerprint: 9D539D90D3328DC7D6C8D3B9D8FF8E1F7DF8B07E
wget -q https://repos.influxdata.com/influxdata-archive_compat.key
echo '393e8779c89ac8d958f81f942f9ad7fb82a25e133faddaf92e15b16e6ac9ce4c influxdata-archive_compat.key' | sha256sum -c && cat influxdata-archive_compat.key | gpg --dearmor | sudo tee /etc/apt/trusted.gpg.d/influxdata-archive_compat.gpg > /dev/null
echo 'deb [signed-by=/etc/apt/trusted.gpg.d/influxdata-archive_compat.gpg] https://repos.influxdata.com/debian stable main' | sudo tee /etc/apt/sources.list.d/influxdata.list
sudo apt-get update && sudo apt-get install telegraf`

Remove the telegraf config file at `/etc/telegraf/telegraf.conf` and replace it with the one here (link TBA).

To set up the intel stats collector, we need to also add the script that telegraf calls. I use a slightly modified script to Brian Miller, see link here.
Store the script at /opt/get_intel_gpu_status.sh

We will need to run the telegraf service as a root service, otherwise this won't work:
`systemctl stop telegraf`
`systemctl edit telegraf`

Add the following lines:
`[Service]`
`User=root`

Then:
systemctl start telegraf

Keep in mind that influx will need the telegraf-igpu created for this to work.

You should see stats being pulled in now to grafana.



