---
title: "10. LTE Fallback"
has_children: true
nav_order: 10
---

TO COMPLETE AND TEST...

## 10. Mikrotik LTE Fallback with OPNsense

It has happened a few times for me that due to works in the area, the OrangeFR fibre service is offline (for up to 2 months!). In cases like these for the higher plans OrangeFR offers 200Gb per month on your Orange mobile service to use as a hotspot (in addition to your normal allowance). While this may be satisfactory for the general population, I prefer to have an automatic failover in case the fibre is offline, with all WAN traffic still routed through OPNsense and distributed as per normal to my LAN.

This section describes the setup of a Mikrotik wAP LTE device with an Orange mobile “multi-sim” offer which allows sharing of data between the mobile phone SIM card and a second data SIM card. The general intent is to have OPNsense failover to the LTE connection when the fibre is offline. Optionally some services can be severed or firewalled to prevent exceeding data caps.

The only intervention required will be contacting Orange customer service to obtain the 200Gb allowance per month (which can be done via DM on their twitter support). I've only seen once where they automatically provide it due to a big problem on their part.

Mikrotik LTE wAP configuration: LTE connection

Insert the Orange SIM into the Mikrotik unit. Make sure you call OrangeFR and have them activate the SIM card. Dial 740 from an Orange phone, key 1, then key in your 10 digit client number which can be found on your invoice. The SIM is activated instantly afterwards.

Log onto the Mikrotik wifi and access the web interface on 192.168.88.1, user: admin, there is no password.

At the quick setup page, change the country to France, add your SIM PIN (should be 0000 normally) click apply, then go to the webfig page. 

Go to Interfaces -> LTE tab -> APN. Make sure you delete any existing APNs except for the “default”. 

Go back to the LTE tab, and click the button that says “LTE APNs”.

Edit the “default” APN with the following values:
* Name: Orange F
* APN: orange.fr
* IP Type: IPv4
* Use Peer DNS: Yes (checked)
* Add default route: Yes (checked) -- to test
* Default Route Distance: 2
* IPv6 Interface: none
* Authentication: PAP
* User: orange
* Password: orange
* Passthrough Interface: none -- to update

Now telnet into the Mikrotik interface, and enter the MCC and MNC together:

`telnet -l admin 192.168.88.1`

`/interface lte set lte1 operator=20801`




