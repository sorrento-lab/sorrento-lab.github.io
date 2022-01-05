---
title: "80. Fibre Attenuation and WAN Testing"
has_children: true
nav_order: 80
---

During the setup of my SFP fibre connection and ONT testing I was made aware of the importance of the fibre connection attenuation by Marcin at LeoLabs. Further research on LaFibre led me to discover the permitted attenuation values for OrangeFR installers [in this very old post](https://lafibre.info/orange-les-news/attenuation-maximum-orange-ftth/msg67521/?PHPSESSID=qn50mcnh4n9969g2dlc3u6i54b#msg67521) - which was -27 dBmin 2012. I have not been able to find other concrete refererences, but it would not surprise me if this was revised to the Class B+ limit of -28 dBm or slightly above it at -27.5 dBm.

Testing the Leox ONT revealed that under some circumstances under load I would have dropped packets. The reason for this is the attenuation of my line at my apartment (lets call it Apartment#2), was -26.9 dBm measured by my Livebox, just inside the 2012 Orange limit. Nevertheless this was not acceptable for the Leox (Marcin indicated that generally for LeoLabs they prefer -25 dBm or higher for optimum connection quality). At the time of my initial testing 6 months prior to running the tests in this section, I did not test the Livebox5 properly. The TP-Link ONT on hand passed all connection tests, however is not able to reach full 1G speed due to firmware bugs. Finally later on when I tested my SFP ONT, it was in the middle of the road, achieving full 2G speeds, however showing some minor packet loss under load.

Naturally now that I have a range of ONTs, I ask the question of what is best for my current use case at Apartment#2?

This section covers the testing of different ONTs to determine which one is best for real world situations for my context at Apartment#2 and its associated line quality (which may not necessarily mean the fastest as per Ookla's speedtest!). This will also provide the method and tools used for future rented apartments (#3 and beyond!) to assess which ONT is best.

My hypothesis is that the TP-Link Class C+ ONT will be best for poor quality lines, and the SFP ONT (Class B+) best for "clean" lines. For Apartment#2 it is difficult to predict if the SFP ONT packet loss performance compromise for poor quality lines will be small enough to justify using its higher speeds over the TP-Link ONT. The testing results should answer this concern.
