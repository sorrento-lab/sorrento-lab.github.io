---
title: 6.4 OPNsense Other
parent: 6. OPNsense Configuration
has_children: false
nav_order: 4
---

## 6.4 OPNsense Other things, small bits, etc

### QEMU Guest Agent
If running under proxmox, add the QEMU guest agent to OPNsense to let Proxmox query OPNsense functionality:
* System: Settings: Tunables -> Add
* Tunable: virtio_console_load, Value: YES
* Reboot
* Services: QEMU Guest Agent: Settings
* Enable service
* Reboot again for good measure
[Reference here (reply#2)](https://forum-opnsense-org.translate.goog/index.php?topic=23284.0&_x_tr_sl=auto&_x_tr_tl=en&_x_tr_hl=en-US&_x_tr_pto=nui)
[Github bug closed here, optional reading](https://github.com/opnsense/plugins/issues/2405)

