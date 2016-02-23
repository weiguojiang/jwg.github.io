---
layout: post
title:  SDN possible usage in data center
category: project
description:  analyze data center switch architecture
---

## ** introduction **
analyze data center switch architecture, then do how SDN can use


## ** web list **
 * [what-is-networking-switch-fabric/](https://www.sdxcentral.com/resources/sdn/what-is-networking-switch-fabric/)
 * [introducing-data-center-fabric-facebook-data-center-network/](https://code.facebook.com/posts/360346274145943/introducing-data-center-fabric-the-next-generation-facebook-data-center-network/)
 * [Technology Overview for Ethernet Switching Fabric](http://www.championsg.com/blog/wp-content/uploads/2013/08/Gartner_technology-overview-for-Ethernet-evolution.pdf)
 * [MC-LAG](https://sites.google.com/site/amitsciscozone/home/link-aggregation/multichassis-link-aggregation-group)
 * [wiki-lag](https://en.wikipedia.org/wiki/MC-LAG)
 * []()


## ** general fabric **
Folded Clos-fabrics are commonly deployed as either layer 2 or layer 3 fabric, with layer 3 fabric being more commonly used due to better scaling properties.
Layer 2 fabric relies on pure L2 switching and thus is limited by traditional L2 switching scaling for cloud tenant traffic. VLAN range barrier determines the maximum amount of VLANs available.
No more than 4094 unique VLANs are available, no loop protection without traditional Spanning Tree Protocols (STP) that do not allow the use of ECMP.
Additional technologies to create ECMP paths have been developed such as Transparent Interconnect of Lots of Links (TRILL, RFC 5556 & RFC 6325), Shortest Path Bridging (SPB, IEEE 802.1aq) and several vendor specific proprietary solutions usually switch stacking.
Both of TRILL and SPB require new L2 headers which mean developing compatible switching hardware and even the vendor proprietary solutions still do not solve the VLAN scalability issue and thus these technologies have not attracted much interest in large scale deployments.
ECMP via Link Aggregation Protocol has been the only viable way to realise a L2 ECMP fabric and that is suited for small scale deployments with few tens of switches.


Layer 3 fabrics are based on IP routing, with build-in loop prevention and ECMP forwarding.
IP routing scales well depending on used protocol from tens to millions of prefixes.
The main protocols used today are OSPF and BGP with some vendor specific protocols like at Google data centers.
Best example in IP routing scaling capability is the Internet itself, it uses BGP to cope with close to 500 000 prefixes in the full Internet table.
