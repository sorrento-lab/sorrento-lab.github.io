---
title: "10. LTE Fallback"
has_children: true
nav_order: 10
---

TO COMPLETE AND TEST...UNTESTED CONFIGURATION!

## 10. Mikrotik LTE Fallback with OPNsense

It has happened a few times for me that due to works in the area, the OrangeFR fibre service is offline (for up to 2 months!). In cases like these for the higher plans OrangeFR offers 200Gb per month on your Orange mobile service to use as a hotspot (in addition to your normal allowance). While this may be satisfactory for the general population, I prefer to have an automatic failover in case the fibre is offline, with all WAN traffic still routed through OPNsense and distributed to selected clients in my LAN.

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

In the firewall rules, add the following rule on the Mikrotik_LAN interface:
Pass / IPV4 / Source: Mikrotik_LAN / Destination: Any
This will allow internet access on the LAN interface to update routerOS.

You should be able to access the Mikrotik_LAN interface now by navigating to 172.16.20.3.

Now to clean it all up:
* Interfaces -> Interface: Disable wireless card (so there is not a stray wifi SSID, you can access via LAN now).
* Interfaces -> Interface: Disable the bridge (no need to remove it)

# Mikrotik LTE wAP configuration: LTE connection

Go to interfaces -> LTE and then click the default "lte1" interface that is available:
* Add your SIM PIN (should be 0000 normally)
* Add the Operator - for Orange, this is the MCC and MNC put together: 20801 (previously I used `/interface lte set lte1 operator=20801`)
* Click Apply and OK

Go to Interfaces -> LTE tab -> APN button. Make sure you delete any existing APNs except for the “default”. 

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

When you click apply, the lte interface will reboot, after 30 seconds or so you will see an LTE WAN IP address available.

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

It is highly recommended to update the LTE firmware - it is tricky to do this without the LTE connection, as the Mikrotik command uses the LTE data to pull the firmware file and install it. To check the firmware:

Go to the terminal of the Mikrotik device. To check the firmware status:
`/interface lte firmware-upgrade lte1`

The response will show you the currently installed and latest available.

To update:
`/interface lte firmware-upgrade lte1 upgrade=yes`

This will initiate the update, it takes about 3-5 mins and there is no install progress bar, do not power cycle the device.

Once installed, the lte interface will reboot and you will have your IP back. At this point it is prudent to disable the lte interface until we are ready to use it again.

# OPNsense configuration

Prior to adding the LTE_WAN into opnsense, ensure that your fibre takes priority. Go to gateways, and for your IPV4 and IPV6 connection set them as priority 254 (instead of 255). Also check the box indicating "upstream gateway" for the IPV4 fibre connection. This will make opnsense use the fibre connection in preference to the LTE_WAN connection.

The LTE connection will be passed via a VLAN, similar to the Orange Fibre to the OPNsense system. For this we use the previously created VLAN 932.

On your Mikrotik LTE wAP, go back to the APN settings, and change:
Passthrough Interface: LTE_VLAN_932
Passthr. MAC Address: generate a random MAC

Note that you will lose the WAN IP address in the Mikrotik interface, as the DHCP request to estabolish WAN connectivity is now expected from OPNsense via vlan 932.

Go to OPNsense GUI. 

Ensure that an interface is available with vlan932 - go to this interface. In my case, I have an interface called `WAN_LTE`

Go to Interfaces -> WAN_LTE, Use the following settings:
•	Enable Interface: YES
•	Prevent Interface Removal: YES
•	Block private networks: YES
•	Block bogon networks: YES
•	IPv4 configuration type: DHCP
•	MAC address: enter the MAC you randomly generated earlier from Mikrotik. This step is important as this will ensure it looks like the MAC that is requested comes from the device holding the SIM itself.
Everything else can stay as default.

Save and apply, and now we should have a mobile WAN address on the `WAN_LTE` interface. In Mikrotik Quickset it will still say `0.0.0.0`, but dig a bit further in the Webfig menus and you will find that Mikrotik creates a sort of bridge with an internal IP address to pass all of this, with the WAN address assigned to it.

----TESTED up to here-----

# OPNsense gateway groupings

To create a failover group, follow this tutorial:
https://docs.opnsense.org/manual/how-tos/multiwan.html

I used cloudflare’s 1.1.1.1 and 1.0.0.1 monitoring DNS instead of googles.

For advanced options:
WAN2 I selected a probe interval of 60 seconds and a time period of 3600. This will mean it pings each minute and averages the results over an hour. As the mobile WAN gateway is the final tier I did not see the sense in constantly pinging and wasting (tiny) amounts of data.

For the primary WAN I left the default probe interval and time period, but I changed the loss interval to 30 seconds.

Note that IPv6 is not monitored. Based on the Orange fibre configuration with opnsense you don’t get a gateway in the gateway menu. Besides – if IPv4 fails, most likely IPv6 will also – although I have seen some uncommon cases where this has not happened!

Make sure you select “Upstream Gateway” for the primary WAN, otherwise both WANs will be presented to the network as gateway candidates and DNS will be half broken. This is not documented in the OPNsense wiki! Furthermore, ensure you update the priority of the primary WAN to be higher than the secondary WAN.






# Untested:
There is a reported bug (as per the Mikrotik forum) of the RP filter erroneously filtering traffic, change it by doing: IP -> Settings -> RP filter: Loose 

Further settings to enable LAN access:
IP-> Routes: Add a new route, only add 172.16.10.1 as a gateway, everything else can stay default.

Return to your home LAN. Check if 172.16.10.3 is accessible. If it is, you can disable the wireless under Interfaces.











