---
layout:     post
title:      ovs feature work
category:   cloud
description:  ovs feature work for RCP
---

### later check

 * ovs not support for the multicast aspects of VXLAN.
 * LY6 & LY8 must use signle IP multicast group for all VXLANs.

 * whether AVS support vlxan overlay ?
 * whether ovs support IP de/fragment ?
 * check legacy feature support about transport level ?
 * check legacy network service mapping with  SDN network ?  
 * check whether ovs support vlan stack or qinq ?

### Q
 * check whether ovs support vlan stack or qinq ?  
   in the feature list which ovs support, it doesn't include 802.1ad  
   but it seems it has patch to support vlan stack.
