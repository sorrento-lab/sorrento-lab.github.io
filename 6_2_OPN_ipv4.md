---
title: 6.2 OFR IPv4 and IPv6
parent: 6. OPNsense Configuration
has_children: false
nav_order: 2
---

### 6.2 OFR IPv4 and IPv6 setup in OPNsense

System -> Settings -> General:  
* DNS servers `1.1.1.1` and `1.0.0.1`
* DNS server options "Allow DNS server list to be overridden by DHCP/PPP on WAN" **UNCHECKED**

System -> Settings -> Miscellaneous:  
* Cryptography Parameters: Hardware acceleration "AES-NI CPU-based acceleration (aesni)"

Interfaces -> Other types -> VLAN. Add the following vlans (shown here bxe0 is the port plugged with the ONT, bxe1 will be our LAN port):

|Interface | VLAN | VLAN priority | Description |
|----------|------|---------------|-|
| bxe0     | 832  | 0             | Orange FTTP DHCP VLAN|
| bxe0     | 838  | 4             | Orange TV VLAN 838 |
| bxe0     | 840  | 5             | Orange TV VLAN 840 |

Note that some older guides on lafibre will indicate that VLAN 835 should also be set up for PPPoE - this is no longer needed, OrangeFR appears to have decommissioned the PPPoE authentication for consumer plans and also appears to be moving to DHCP for Pro plans.

Interfaces -> Assignments: Assign `vlan832 on bxe0` to WAN and then "Save"

Interfaces -> WAN
* Enable interface = ON
* Prevent interface removal = ON
* Block private networks = OFF
* Block bogon networks = ON
* IPv4 config type = DHCP
* IPv6 config type = DHCPv6
* DHCP client config mode = Advanced
  * Override MTU = ON
  * Protocol timing = OPNsense default
  * Lease requirements -> Send options  
  `dhcp-class-identifier "sagem", user-class "+FSVDSL_livebox.Internet.softathome.Livebox4", option-90 $OPTION90` note that the livebox5 still sends a livebox4 user-class. $OPTION90 is discussed above.  
  * Lease requirements -> Request options  
  `subnet-mask, broadcast-address, dhcp-lease-time, dhcp-renewal-time, dhcp-rebinding-time, domain-search, routers, domain-name-servers, option-90`
  * Option modifiers
  `vlan-parent "bxe0", vlan-id 832, vlan-pcp 6`
  
* DHCPv6 client config mode = Advanced
  * Use VLAN priority = Internetwork Control (6)
  * Interface statement -> Send options  
  `ia-pd 0, raw-option 6 00:0B:00:11:00:17:00:18, raw-option 15 00:2b:46:53:56:44:53:4c:5f:6c:69:76:65:62:6f:78:2e:49:6e:74:65:72:6e:65:74:2e:73:6f:66:74:61:74:68:6f:6d:65:2e:6c:69:76:65:62:6f:78:34, raw-option 16 00:00:04:0E:00:05:73:61:67:65:6D, raw-option 11 $OPTION90`
  * Interface statement -> Request options can remain blank
  * Identity association -> Prefix delegation = ON
  * Identity association -> id-assoc pd ID = 0
  * Prefix interface ->	Prefix Interface Site-Level Aggregation Length = 8

Interfaces -> Settings
* Ensure all disable hardware acceleration checkboxes are "ON" i.e. hardware acceleration disabled
* Disable VLAN hardware filtering
* Suppress ARP messages = ON
* IPv6 DHCP
  * Prevent release = ON
  * Log level = Standard
  * DHCP unique identifier = `00:03:00:01:+MAC address` See option 61 description above. Note that here there is an additional `00:03` added to the front.

Make sure to click save and apply at each screen.

References:
[https://docs.opnsense.org/manual/how-tos/orange_fr_fttp.html](https://docs.opnsense.org/manual/how-tos/orange_fr_fttp.html)
