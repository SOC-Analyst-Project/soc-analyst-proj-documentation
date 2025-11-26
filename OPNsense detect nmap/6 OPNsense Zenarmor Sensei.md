https://docs.opnsense.org/vendor/sunnyvalley/zenarmor_install.html

We need Layer 2 mode configured for Zenarmor to detect ARP scans. Layer 2 mode requires us to create an optional interface (OPT1), and connect it with LAN via a bridge in OPNsense. A bridge will behave like a switch, all the traffic in LAN (except known unicast) are broadcasted and forwarded to the OPT1 via the bridge.

> A **bridge** makes multiple NICs behave like **switch ports**

>Normal LAN = only sees IP packets.  
>Bridge = sees all Ethernet frames (ARP, broadcasts, MAC).

>**the LAN NIC delivers ARP frames to the kernel, not to Zenarmor.**  
>By the time Zenarmor gets traffic, the ARP frame is already consumed by the OS and no longer available.

