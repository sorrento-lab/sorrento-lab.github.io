---
title: "10. LTE Fallback"
has_children: true
nav_order: 10
---

TO COMPLETE AND TEST...UNTESTED CONFIGURATION!

## 10. Mikrotik LTE Fallback with OPNsense

It has happened a few times for me that due to works in the area, the OrangeFR fibre service is offline (for up to 2 months!). In cases like these for the higher plans OrangeFR offers 200Gb per month on your Orange mobile service to use as a hotspot (in addition to your normal allowance). While this may be satisfactory for the general population, I prefer to have an automatic failover in case the fibre is offline, with all WAN traffic still routed through OPNsense and distributed as per normal to my LAN.

This section describes the setup of a Mikrotik wAP LTE device with an Orange mobile “multi-sim” offer which allows sharing of data between the mobile phone SIM card and a second data SIM card. The general intent is to have OPNsense failover to the LTE connection when the fibre is offline. Optionally some services can be severed or firewalled to prevent exceeding data caps.

The only intervention required will be contacting Orange customer service to obtain the 200Gb allowance per month (which can be done via DM on their twitter support). I've only seen once where they automatically provide it due to a big problem on their part.

# Configuring LAN access in Mikrotik wAP

Insert the Orange SIM into the Mikrotik unit. Make sure you call OrangeFR and have them activate the SIM card. Dial 740 from an Orange phone, key 1, then key in your 10 digit client number which can be found on your invoice. The SIM is activated instantly afterwards.

Log onto the Mikrotik wifi and access the web interface on 192.168.88.1, user: admin, there is no password.

At the quick setup page, enter a new Mikrotik interface password, change the wifi country to France, and add a wifi password.

At this point you will need to re-connect to the wifi.

Still in the Quick-setup page, ensure you are on the LTE AP setup. Disable the "firewall router" box, disable DHCP server and disable NAT. Keep the default LAN as 192.168.88.1.

Apply changes and go into webfig. From here on in it is easy to lock yourself out, so use safe mode as required.

In webfig, go to Bridge -> Ports and remove the ether1 port from the bridge. 

Go to Interfaces -> VLAN and create two VLANs:
* LTE_VLAN_932: with vlan 932, this will be our LTE WAN VLAN
* Ether_VLAN_20: this will be our management LAN VLAN

Note that the vlans cannot be on the bridge, and the ether1 interface needs to be removed from the bridge for these two vlans to work. VLAN 1 will not work as the management VLAN.

Go to IP -> Addresses and set a static IP address of 172.16.20.3/24 on Ether_VLAN_20. A network will be automatically populated.

Go to DNS -> set your DNS server. I use 172.16.10.2, OPNsense allows inter-vlan routing by default.

Finally go to routes -> add new route and then add the gateway of 172.16.20.1. This will add another hop and allow everything to work later.

We can leave the Mikrotik for now to configure OPNsense.

# OPNsense LAN configuration

Create two new VLANs in OPNsense:
* VLAN 932: Orange mobile internet
* VLAN 20: Mikrotik LAN

Assign these to your trunk or Mikrotik interface as necessary. Note if using a trunk port that the VLANs need to be configured in the switch as well.

Ensure the assignments are set up and add the interfaces to the list. We will leave the WAN one disabled for now.

In the Mikrotik_LAN interface set "Static IPv4" and "172.16.20.1" as the interface address.



Set your computer to a static 192.168.88.X address to connect to the Mikrotik.

IP->DHCP Server: Disable the DHCP server
IP-> DNS->Click the static button and disable the DNS server
IP-> Addresses: Add a new address inside your LAN (I used 172.16.10.3/24 with a 172.16.10.0 network). You will need to shuffle some things around to maintain connectivity. The Mikrotik wAP does not like showing two interfaces when a VLAN is active on the bridge. Therefore, the bridge cannot be used. Both WAN and LAN interface need to be presented as VLANs on the ethernet interface (e.g. 932 and 10 respectively), and the ethernet interface removed from the bridge. But be careful! You can lose complete connectivity to your Mikrotik as you will lock yourself out if you disable the bridge. Therefore some sort of shuffle needs to take place using the wirelesspart of the Mikrotik wAP. To be detailed later.

Go back to OPNsense on your home network. Add OPT3 as an interface, which is the LAN of the Mikrotik to the LAN bridge. 

Go back to the Mikrotik network, you should be able to access it with a static 172 address now.


# OPNsense configuration

The LTE connection will be passed via a VLAN, similar to the Orange Fibre to the OPNsense system.

In OPNsense: Interfaces -> VLAN Tab -> Add New

Name: mobile_WAN
VLAN ID: 932 (can be anything)
Interface: ether1
Click Save/OK

On your Mikrotik LTE wAP, go back to the APN settings, and change:
Passthrough Interface: mobile_WAN
Passthr. MAC Address: generate a random MAC

Note that you will lose the WAN IP address in the Mikrotik interface, as the DHCP request is now expected from OPNsense via vlan 932.

For the next step, make sure that the interface you have connected the Mikrotik is passed through to Proxmox. For my case, I had a spare Realtek NIC on my motherboard I passed through. You can do this with some switching and routing, but I prefer all my WAN connections on dedicated passed through devices so that any errors I make will be kept upstream of OPNsense.

Go to OPNsense GUI. 

Add the following:
Interfaces -> Other Types -> VLANs -> Add a vlan on re0 (Realtek) with number 932. Call it Orange Mobile Internet

Interfaces -> Assignments -> Assign vlan932 on re0 named WAN2

Interfaces -> WAN2: Use the following settings:
•	Enable Interface: YES
•	Prevent Interface Removal: YES
•	Block private networks: YES
•	Block bogon networks: YES
•	IPv4 configuration type: DHCP
•	MAC address: enter the MAC you randomly generated earlier from Mikrotik. This step is important as this will ensure it looks like the MAC that is requested comes from the device holding the SIM itself.
Everything else can stay as default.

Save and apply, and now we should have a mobile WAN address on the WAN2 interface.

# OPNsense gateway groupings

At this stage you may lose connectivity due to having two single gateways.

To create a failover group, follow this tutorial:
https://docs.opnsense.org/manual/how-tos/multiwan.html

I used cloudflare’s 1.1.1.1 and 1.0.0.1 monitoring DNS instead of googles.

For advanced options:
WAN2 I selected a probe interval of 60 seconds and a time period of 3600. This will mean it pings each minute and averages the results over an hour. As the mobile WAN gateway is the final tier I did not see the sense in constantly pinging and wasting (tiny) amounts of data.

For the primary WAN I left the default probe interval and time period, but I changed the loss interval to 30 seconds.

Note that IPv6 is not monitored. Based on the Orange fibre configuration with opnsense you don’t get a gateway in the gateway menu. Besides – if IPv4 fails, most likely IPv6 will also – although I have seen some uncommon cases where this has not happened!

Make sure you select “Upstream Gateway” for the primary WAN, otherwise both WANs will be presented to the network as gateway candidates and DNS will be half broken. This is not documented in the OPNsense wiki! Furthermore, ensure you update the priority of the primary WAN to be higher than the secondary WAN.

# Configuring LAN access in Mikrotik wAP

This will configure web interface access over ethernet (remembering that internet is over vlan 932).

Set your computer to a static 192.168.88.X address to connect to the Mikrotik.

IP->DHCP Server: Disable the DHCP server
IP-> DNS->Click the static button and disable the DNS server
IP-> Addresses: Add a new address inside your LAN (I used 172.16.10.3/24 with a 172.16.10.0 network). You will need to shuffle some things around to maintain connectivity. The Mikrotik wAP does not like showing two interfaces when a VLAN is active on the bridge. Therefore, the bridge cannot be used. Both WAN and LAN interface need to be presented as VLANs on the ethernet interface (e.g. 932 and 10 respectively), and the ethernet interface removed from the bridge. But be careful! You can lose complete connectivity to your Mikrotik as you will lock yourself out if you disable the bridge. Therefore some sort of shuffle needs to take place using the wirelesspart of the Mikrotik wAP. To be detailed later.

Go back to OPNsense on your home network. Add OPT3 as an interface, which is the LAN of the Mikrotik to the LAN bridge. 

Go back to the Mikrotik network, you should be able to access it with a static 172 address now.

------
# Mikrotik LTE wAP configuration: LTE connection

add your SIM PIN (should be 0000 normally) click apply, then go to the webfig page. 

Go to Interfaces -> LTE tab -> APN. Make sure you delete any existing APNs except for the “default”. 

Go back to the LTE tab, and click the button that says “LTE APNs”.

Edit the “default” APN with the following values:
* Name: Orange F
* APN: orange.fr
* IP Type: IPv4
* Use Peer DNS: Yes (checked)
* Add default route: Yes (checked)
* Default Route Distance: 2
* IPv6 Interface: none
* Authentication: PAP
* User: orange
* Password: orange
* Passthrough Interface: none

Now telnet into the Mikrotik interface, and enter the MCC and MNC together:

`telnet -l admin 192.168.88.1`
`/interface lte set lte1 operator=20801`

You can monitor the connection link from here:
`/interface lte info lte1`

If it connects correctly you should see:
`           pin-status: ok`
`  registration-status: registered`
`        functionality: full`
`         manufacturer: "MikroTik"`
`                model: "R11e-LTE"`
`             revision: "MikroTik_CP_2.160.000_v015"`
`     current-operator: Orange F`
Followed by some cellular information and numbers.

At this point it is prudent to disable the LTE interface while we setup the bridging (Interfaces, click the little “D” next to LTE).

# Untested:
There is a reported bug (as per the Mikrotik forum) of the RP filter erroneously filtering traffic, change it by doing: IP -> Settings -> RP filter: Loose 

Further settings to enable LAN access:
IP-> Routes: Add a new route, only add 172.16.10.1 as a gateway, everything else can stay default.

Return to your home LAN. Check if 172.16.10.3 is accessible. If it is, you can disable the wireless under Interfaces.











