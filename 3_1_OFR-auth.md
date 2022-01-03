---
title: 3.1 OrangeFR Auth Parameters
parent: 3. Preparation work for OFR
has_children: false
nav_order: 1
---

### 3.1 Orange Authentication Parameters
This section presents in brief the authentication steps to achieve a WAN connection - further reading and details can be found in the links at the end of this sub-section. This is my amateur non-expert understanding.

OrangeFR distributes its internet connection by a GPON fibre network. The fibre connections to each distribution point are managed by Optical Line Terminals (OLTs). Each end user connects to the OLT via Optical Network Termination (ONTs, also known as Optical Network Units - ONUs). This forms the first step of achieving a network connection with Orange. The ONT has a discrete microprocessor which will negotiate a connection on the OLT "tree" using certain authentication parameters. This means that it is not possible to have two ONTs with the same authentication parameters attempt to synchronise to the OLT - the OLT will not allow it. ONTs come in SFP forms, or larger forms taking a fibre connection and outputting a RJ-45 1Gbps connection . More can be read on my experiences with these devices at Section ==TBC==.

As of writing and my experience, OrangeFR requires the following ONT parameters to authenticate with an Orange OLT:
* ONT serial number - this can be found on the label underneath your livebox, or in the admin section of your livebox at `192.168.1.1`.
  * The ONT serial number is composed of two parts. The first is the vendor identifier. Typically for Livebox5 users it will be `SMBS`, which is a plain text implementation. Then, there are 8 digits following the 4 letters which are in hexadecimal format. 
* ONT HardwareVersion - this can be found in the admin section of your livebox at `192.168.1.1`.
  * This also contains the 4 letters from the hardware vendor `SMBS`, as well as 8 alphanumeric letters.

Livebox4 users may also need SLID - this is out of the scope of this journal.

There is some argument for replicating as many parameters as possible to make your ONT look as much as possible as a Orange ONT, however from what I have seen there have been no reported experiences of users being kicked off OLT trees for not enough authentication parameters. 
The available ONT details in a Livebox5 ONT section (along with their expected values):
* VeipPptpUni (TRUE)
* Omci Is TmOwner (FALSE)
* Max Bit Rate Supported (1mW)
* Signal RxPower (value in dBm, typically -15 to -25. Limit <-28dBm for a B+ transceiver, however this is not ideal and could lead to dropped packets under load)
* Signal TxPower (range between +1.5 and +5 dBm)
* Temperature (of ONT transceiver package, in degC)
* Bias (integer)
* Serial number (SMBS followed by 8 digits in hex, i.e. 4 bytes)
* Hardware Version (SMBS followed by 8 alphanumerical digits)
* Equipment ID (SagemcomFast5656OFR)
* Vendor ID (SMBS)
* Vendor Product Code (1)
* Registration ID (mine shows N/A)
* Voltage (mine shows 32.9V)
* ONT Software Version 0 (SAHEOFR followed by 6 numbers)
* ONT Software Version 1 (SAHEOFR followed by 6 numbers)
* ONT Software Version active (0 or 1)

The second step of authentication with OrangeFR is using DHCP. PPPoE is being slowly taken out of service by OrangeFR, as evidenced in links seen in forum posts on LaFibre.

OrangeFR distributes its DHCP internet connections via VLANs, in particular:
* VLAN 832: DHCP WAN + IP-TV VOD
* VLAN 835: PPPoE WAN (no longer in use)
* VLAN 838: IP-TV VOD - however this is not used in my implementation, see below
* VLAN 840: IP-TV

According to the [LaFibre OPNsense OFR discussion](https://lafibre.info/remplacer-livebox/opnsense-remplacer-livebox-aucune-modification-necessaire/msg586096/#msg586096) VLAN 838 is still in service but seems to no longer be the default that is used. This can also be observed in the Livebox5 admin info section. My implementation uses VLAN 832 and 840 for the OrangeTV service.

OrangeFR DHCP authentication is a little trickier than PPPoE authentication. Unlike PPPoE, which needs a simple username + password, DHCP authentication requires sending and receiving sets of characters with the DHCP process called "options". The process for IPv4 and IPv6 is slightly different for each. The actual input of the options is covered in the OPNsense section (Section ==TBC==). 

For information I have added brief descriptions for each option:
IPv4 sent options (sent by the livebox):  

|Option number| Description | Value | Remarks| Source of data |
|-------------|-------------|-------|--------|----------------|
| 60          | Class ID    |`sagem`|vendor of livebox|Found via liveboxtools `option 60 = 736167656d` (in hex)|
| 61          | Client ID | `019x9x9x9x9x9x`|"01" followed by MAC address of livebox. In OPNsense this is entered in the DHCP unique identifier field, not in the sent options. | On back of livebox or via livebox tools, substitute 9x9x9x9x9x9x with your MAC|
| 77          | User class ID | `+FSVDSL_livebox.Internet.<br/>softathome.Livebox4` |Livebox4 is shown also in Livebox5 models|Found via liveboxtools option 77 = `2b46535644534c5f6c697665626f782e496e7465726e65742e736f66746174686f6d652e4c697665626f7834` (in hex)|
| 90          | Authentication |140 char string|see below|  |

Option 90 is orangeFR's authentication method. It uses the following method to generate the 140 character string:
* The first 11 bytes are set to 0: this is a location the size of the standard DHCP Option 90 header, as defined in RFC 3118 . These parameters not being used, they are set to 0.
* Then it is a sequence of TLV (type-length-value) fields. In other words, it is a sequence of proprietary parameters defined by Orange which follow this pattern: 1 type byte, 1 size byte (including type, size and value bytes), followed by a certain number of value bytes based on size.
  * First there is this hard-defined sequence of bytes in the binary, I don't know what it is: 1a0900000558 (so 1a is the type and 09 the size)
  * Then another fixed field whose value is "A", maybe a version of format ...: 010341 (01 is the type, 03 is the size, 41 is the value "A" in ASCII)
  * Then a field whose type is strangely also 01, and the value is the Orange username. It takes the format of 010d followed by `fti/abcdefg` converted to hex: 010dXXXXXXXXXXXXXXXXXXXXXX. You will need to get your orange username and password from orange. OrangeFR twitter support is helpful here.
  * Then a field whose type is 3c12, followed by 16 random bytes (let's call them RANDSTRING) is generated by the DHCP client: 3c12XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX
  * Finally, a field whose type is 0313, and which contains 17 bytes of value:
    * 1 random byte, let's call it RANDCHAR, immediately followed by
    * 16 bytes which is an MD5 hash generated with the following formula: MD5 (RANDCHAR + PASSWORD + RANDSTRING), where PASSWORD is the OFR fti password

[For further reading on Option 90, reference to this post. The information above is from post #10.](https://lafibre.info/remplacer-livebox/cacking-nouveau-systeme-de-generation-de-loption-90-dhcp/)

LaFibre's "zoc" created a script to develop the long option 90, see reference xx  

    #!/bin/bash
    
    login='fti/*******'
    pass='****************'

    tohex() {
      for h in $(echo $1 | sed "s/\(.\)/\1 /g"); do printf %02x \'$h; done</br>
    }
    
    addsep() {
      echo $(echo $1 | sed "s/\(.\)\(.\)/:\1\2/g")</br>
    }
    
    r=$(dd if=/dev/urandom bs=1k count=1 2>&1 | md5sum | cut -c1-16)
    id=${r:0:1}
    h=3C12$(tohex ${r})0313$(tohex ${id})$(echo -n ${id}${pass}${r} | md5sum | cut -c1-32)
    
    echo 00:00:00:00:00:00:00:00:00:00:00:1A:09:00:00:05:58:01:03:41:01:0D$(addsep $(tohex ${login})${h})

IPv4 requested options (requested by the livebox):
* 1: Subnet mask
* 3: Router address
* 6: DNS server addresses
* 15: Domain name of client
* 28: Broadcast address
* 51: IP address lease time
* 58: DHCP renewal time
* 59: DHCP rebinding time
* 90: Authentication
* 119: DNS domain search list
* 120: SIP servers DHCP option (used for VOIP)
* 125: Vendor-Identifying Vendor-Specific Information

IPv6 sent raw options (note orangeFR uses "raw" options):
* 6: ORO (Option Request Option) which is used to identify a list of options needed from the server
* 11: Authentication. For OFR this is the same string as DHCPv4 Option 90
* 15: User class
* 16: Vendor class
* 17: "IPv6 requested" - ==to be investigated, my config does not need this raw option...==

IPv6 received raw options:
* 1: Client ID
* 2: Server ID
* 11: Authentication
* 25: Identity association (prefix)

Further info on DHCP options can be found here:
DHCPv4: https://www.iana.org/assignments/bootp-dhcp-parameters/bootp-dhcp-parameters.xhtml
DHCPv6: http://www.networksorcery.com/enp/protocol/dhcpv6.htm

Finally note that for certain areas in france the DHCP options need to be sent via a certain VLAN priority, otherwise they will be ignored. The priorities for selection are either 0 or 6 (VLAN DHCP priority 6 appears in use for Paris area).

For TV authentication:
* Option 125 is sent **from the router to the OFR-TV unit** via the dummy DHCP server that will be set up. 
* Orange DNS server list
* Domain search list for your geographical area

References 
https://lafibre.info/remplacer-livebox/cacking-nouveau-systeme-de-generation-de-loption-90-dhcp/588/
https://lafibre.info/remplacer-livebox/opnsense-remplacer-livebox-aucune-modification-necessaire/
https://docs.opnsense.org/manual/how-tos/orange_fr_fttp.html
https://docs.opnsense.org/manual/how-tos/orange_fr_tvf.html
https://lafibre.info/remplacer-livebox/guide-de-connexion-fibre-directement-sur-un-routeur-voire-meme-en-2gbps/
(If I have shown something here that has not been referenced properly please let me know!)
