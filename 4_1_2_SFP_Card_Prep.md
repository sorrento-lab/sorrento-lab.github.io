---
title: 4.1.2 BCM57810S SFP+ card preparation
parent: 4.1 SFP ONT
grand_parent: 4. ONT Preparation
nav_order: 2
---

## 3.2 BCM57810S SFP+ card preparation

The Dell N20KJ card comes pre-configured as a 1/10 Gbps card. On DSL reports, it was worked out that the BCM57810S chipset can activate 2.5G and 5G modes.

To do this, a diagnostics tool must be downloaded and used on the card.

1. Download the ediag tool.  
The link used for my card is [here](https://mega.nz/file/b2YWHahJ#R8J-sEzQ5wm9EMxlyu4AULj5JqadlnJsc0zkfeIu57U)
2. Download rufus from rufus.ie
3. Create a MS-DOS bootable USB, follow the guide [https://www.howtogeek.com/136987/how-to-create-a-bootable-dos-usb-drive/](https://www.howtogeek.com/136987/how-to-create-a-bootable-dos-usb-drive/)
4. Find the ediag.exe file, and then copy it and all the contents of the folder in which it resides to the USB
5. Boot from the USB on the PC where the card is plugged in. You should reach a DOS prompt.
6. Access ediag.exe in engineering mode by running `ediag.exe -b10eng`
7. Execute the following commands (these may differ from card to card):  
`device 1` to select the first port where the ONT will reside  
`nvm cfg` to access NVRAM configurations settings  
`7` Link settings  
`35=70` Speed capability mask for P0 = 0x10 for 1 Gbps + 0x20 for 2.5 Gbps + 0x40 for 10 Gbps = 0x70  
`36=70` Speed capability mask for P3 = 0x10 for 1 Gbps + 0x20 for 2.5 Gbps + 0x40 for 10 Gbps = 0x70  
`56=6` Link speed = 2.5G  
`59=6`  
`save`  
`exit`  

If you are using a custom fan, sometimes it will trigger a fan fail warning on the SFP+ card, and shut the card down. To bypass the fan cut-off, you can do the following:  
`device 1` it does not matter which device is selected for this step, it will affect both ports  
`nvm cfg` to access NVRAM configurations settings  
`3` Board I/O (if I remember correctly)  
`83=1` to bypass the fan fail warning. Note that this fan fail warning will shut down the board unless it is bypassed.  
`save`  
`exit`  

This completes the SFP+ card preparation.
