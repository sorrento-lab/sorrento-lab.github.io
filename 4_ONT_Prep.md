---
title: "4. ONT Preparation"
has_children: true
nav_order: 4
---

### 4. ONT Preparation

This section details the preparation of the Optical Network Terminal/Unit (ONT/ONU) that I have tried.

There are two key differences in the ONT physical profiles that are available on the market. The SFP profile is essentially an SFP transceiver with a built in microprocessor that will plug into an SFP(+) card and are powered by the card. The standalone RJ-45 units are externally powered, and contain an optical port and RJ-45 (typically 1Gbps, although LeoLabs is developing a 2.5Gbps version).

ONTs will be constructed to respect a receiver and transmitter sensitivity and attenuation requirements. The majority of the ONTs on the market in both physical formats will respect a B+ class requirement (receiver sens. <-28 dBm). A small amount of ONTs have been built to allow higher attenuation values (class C+, <-32 dBm). [A good summary can be found at this site](https://www.multicominc.com/training/technical-resources/sfps-b-and-c-whats-the-difference/). 

Once the fibre line attenuation values begin drifting towards the receiver limit, packets will begin to be dropped, especially under load. Generally ISPs will respect B+ requirements, however in certain edge cases (such as my current apartment!) the line attenuation is close to the limit of B+, but not exceeding, and for the ISP this is still acceptable as the ISP provided box onboard diagnostics will still show "healthy". Further commentary and testing can be found in Section ==TBC==.

I have tested the following ONTs (detailed in the sub-sections):
* Nokia-Alcatel G-010S-A (Class B+) activated to HSGMII 2.2/1.25Gbps mode with Broadcom BCM57810S SFP+ PCIe card (activated to 2.5Gbps mode)
* TP-Link TX-6610 1Gbps (Class C+) RJ-45
* LeoLabs Leox 1Gbps (Class B+) RJ-45

The ONTs themselves perform the function of authenticating themselves and placing themselves on the OLT tree using the ISP provided serial number and HardwareVersion variables.

Finally it should be noted that some branded ONTs will only synchronise to certain brands of OLTs. The OLTs in usage at OFR appear to be of three different varieties:
* Huawei nice
* Huawei not-so-nice
* Nokia-Alcatel

[Further info on OLT acceptability of ONTs can be found in @dmfr's post here.](https://lafibre.info/remplacer-livebox/guide-de-connexion-fibre-directement-sur-un-routeur-voire-meme-en-2gbps/msg894389/#msg894389)

Further SFP ONT testing can be found at [the excellent 2Gbps LaFibre thread here, look for @Gnubytes first posts.](https://lafibre.info/remplacer-livebox/guide-de-connexion-fibre-directement-sur-un-routeur-voire-meme-en-2gbps/)

Other RJ-45 ONTs that have been shown to work include:
* Huawei HG8010h (I believe this used to be the ONT supplied by OFR for Livebox3/4 customers)
* Ubiquiti UFiber Nano G [(only compatible with Huawei)](https://lafibre.info/remplacer-livebox/ubiquiti-ufiber-nano-g/)
