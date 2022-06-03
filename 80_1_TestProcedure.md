---
title: 80.1 WAN Attentuation Testing Method
parent: 80. Fibre Attenuation and WAN Testing
has_children: false
nav_order: 1
---

## 80.1 WAN Attentuation Testing Method

I will use some simple tests to check the performance of my WAN connection on the different devices.

1. Speedtest from OPNsense CLI (x5 runs, discard best and worst and average of 3x) - measure speed
2. wget a 5GB file from a EU server which is known to have large bandwidth - measure average speed and DL time
3. Run 5x wget 5GB at the same time - measure average speed and DL time
4. Loaded ping test `ping 195.xxx.xxx.xxx -s 65507` - check ping times, if ping times out, reduce packet payload until 0% loss and then measure times
5. `time dig` test to check DNS response

The above points should give me a basic comparison.
