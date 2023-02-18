---
title: 6.3 OPNsense LAN configuration
parent: 6. OPNsense Configuration
has_children: false
nav_order: 3
---



OrangeTV config

To add OrangeTV to your own wifi:
1. Disable all your wifi networks
2. Add in your livebox and ensure it is online (sometimes it power cycles due to firmware updates if it has been offline for a while)
3. Turn on your decoder, it will ask if you have disconnected the ethernet cable voluntarily. Say yes, then select your livebox model at the prompt.
4. It will then guide you through the wifi pairing procedure. WPS key will need to be pressed on the livebox.
5. Allow it to pair, when you have a stable TV stream, press the return+OK key on your TV remote to enter the Orange maintenance menu
6. Go down to "Wifi/Bluetooth" and take a photo of the SSID
7. Turn off livebox and decoder, bring your system back online
8. Add the SSID to your AP as a new SSID. Ensure it is associated with VLAN 45
9. Turn on your decoder, it should jump on the new SSID straight away!

Reference here: https://lafibre.info/remplacer-livebox/le-guide-complet-pour-usgusg-pro-internet-tv-livebox-ipv6/msg827102/#msg827102
