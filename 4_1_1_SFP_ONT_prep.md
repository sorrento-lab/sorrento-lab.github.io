---
title: 4.1.1 G-010S-A SFP ONT Preparation
parent: 4.1 SFP ONT
grand_parent: 4. ONT Preparation
nav_order: 1
---

## 3.1 SFP ONT preparation

This section details ONT preparation of a Nokia-Alcatel G-010S-A. **Note that the recommended unit on LaFibre is now a [FS.com 133619 GPON ONU](https://www.fs.com/fr/products/133619.html)**.

First, a small hardware modification needs to be carried out on the ONT itself, inspired by [rsaxvc.net](https://rsaxvc.net/blog/2020/8/15/Nokia_G-010S-A_Pin_6_Issue.html). This enables the ONT to boot with the Dell N20KJ card that I am using. 
I ordered a conductive solder pen from amazon. I applied the solder to the resistor block by first using the pen on a piece of paper in a small circle, then taking a toothpick, scooping the solder from the paper and carefully applying it around the resistor block. Here is my attempt: 
IMAGE TBC

Next comes the configuration of the ONT itself. There are multiple ways to boot the ONT and access it. The easiest is to use a TP-Link MC-220L. As my server was not required to be on 24/7, I could use this as a "spare" computer. Plug your SFP+ card into the computer and plug in the ONT into one of the ports.
Now we need to somehow access the ONT sitting inside the SFP+ port. I did not have my SFP+/RJ-45 switch at the time, so here is my over-worked solution: using a linux live USB to create a ethernet/SFP+ bridge, and then using my laptop via the ethernet port to SSH into the ONT. A lot of these steps could be short-cut in hindsight, but here we are!

Use a linux live USB to boot into Ubuntu. Connect an ethernet cable from the livebox5 to install the bridge-utils package manager:
`$ apt-get install bridge-utils`
Once this is installed, run the nm-connection-editor application:
`$ nm-connection-editor`
This will open up the connection editor. You should see your two fibre interfaces, as well as your onboard motherboard ethernet interfaces. 
More is described in [Aaron Kili's Network Bridge Article](https://www.tecmint.com/create-network-bridge-in-ubuntu/), but the general steps you will need to take are:
1. Add a new connection profile, define it as a bridge
2. Add the ethernet port and the fibre port into the bridge group
3. Set a static IP address
4. Remove the slaves from the network configuration so that only the bridge group remains

Connecting your laptop with a static IP in the same subnet as the server PC will allow you to access the ONT on `192.168.1.10`.

SSH into the ONT by:  
`ssh 192.168.1.10 -l ONTUSER`  
at the prompt, password is `SUGAR2A041`  

Update the following parameters:  
`ONTUSER # ritool set MfrID SMBS` where SMBS is the vendor ID we identified in the livebox information  
`ONTUSER # ritool set G984Serial 534d425300000000` where 534d4253 is "SMBS" in hexa, and the rest of the serial number is used as is  
`ONTUSER #ritool set HardwareVersion SMBSSGLBF114` oddly my ONT did not need this converted to hexa - it accepted this value as shown  
`ONTUSER # ritool dump` this writes the values to the ONT  

At this stage software version was not updated (and does not seem required).

`ONTUSER # fw_setenv sgmii_mode 5` To activate the Lantiq chipset 2.5 Gbps mode. Mode 4 is for 1 Gbps.

`ONTUSER # reboot` The reboot cycle takes approx. 60 seconds, comparitavely slower than a TX-6610 or other standalone units.

You can watch the connection status once the ONT boots using: `ONTUSER # watch -n 1 onu ploamsg`  
This should give you a readout as follows: `errorcode=0 curr_state=5 previous_state=4 elapsed_msec=37876`  
As you can see the curr_state is 5, which is authenticated.

The ONT can now be considered ready for use.
