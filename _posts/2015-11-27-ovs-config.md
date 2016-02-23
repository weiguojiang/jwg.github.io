---
layout:     post
title:      ovs
category:   cloud
description: ovs configuration for vlan and vxlan
---

## ** refer link **
 * https://github.com/openvswitch/ovs/blob/master/FAQ.md
 * http://openvswitch.org/support/config-cookbooks/port-tunneling/
 * http://openvswitch.org/support/config-cookbooks/vlan-configuration-cookbook/
 * http://networkstatic.net/setting-overlays-open-vswitch/

## Setup env

<img src="/images/other/2host-4vm-2nic.png" alt="ovs" width="1000"></a>

### Configuration Steps for vlan

in H1

        ovs-vsctl add-br br0
        ovs-vsctl add-port br0 tap0 tag=100
        ovs-vsctl add-port br0 tap1 tag=101
        ovs-vsctl add-port br0 eth0

 in H2  

        ovs-vsctl add-br br0
        ovs-vsctl add-port br0 tap0 tag=100
        ovs-vsctl add-port br0 tap1 tag=101
        ovs-vsctl add-port br0 eth0

`if you add port eth0 into ovs, then you can't config the IP in this port`

### Configuration Steps for GRE tunnel

in H1  

        ovs-vsctl add-br br0
        ovs-vsctl add-port br0 tap0
        ovs-vsctl add-port br0 tap1
        ovs-vsctl add-port br0 gre0 -- set interface gre0 type=gre options:remote_ip=`IP address of eth0 on Host2`

 in H2  

        ovs-vsctl add-br br0
        ovs-vsctl add-port br0 tap0
        ovs-vsctl add-port br0 tap1
        ovs-vsctl add-port br0 gre0 -- set interface gre0 type=gre options:remote_ip= `IP address of eth0 on Host1`

`For tunnel, if the IP is the eth0, then you can't add this port into ovs`

### Configuration Steps for vxlan tunnel

in H1  

 * ovs-vsctl add-br br0
 * ovs-vsctl add-port br0 vxlan -- set Interface vxlan type=vxlan options:key=flow
 ptions:local_ip=10.56.212.53 options:remote_ip=10.56.212.56 ofport_request=10
 * ovs-vsctl add-port br0 tap1 -- set Interface tap1 type=internal
 * ip netns add ns1
 * ip link set tap1 netns ns1
 * ip netns exec ns1 ip link set dev tap1 up
 * ip netns exec ns1 ip addr add 10.0.0.1/24 dev tap1

 in H2

 * ovs-vsctl add-br br0
 * ovs-vsctl add-port br0 vxlan -- set Interface vxlan type=vxlan options:key=flow options:local_ip=10.56.212.56 options:remote_ip=10.56.212.53 ofport_request=10
 * ovs-vsctl add-port br0 tap1 -- set Interface tap1 type=internal
 * ip netns add ns1
 * ip link set tap1 netns ns1
 * ip netns exec ns1 ip link set dev tap1 up
 * ip netns exec ns1 ip addr add 10.0.0.2/24 dev tap1

`For tunnel, if the IP is the eth0, then you can't add this port into ovs`

 in ovs

        ovs-ofctl add-flow br0 "table=0,in_port=1,actions=set_field:103->tun_id,goto_table:1"
        ovs-ofctl add-flow br0 "table=0,actions=goto_table:1"

        ovs-ofctl add-flow br0 "table=1,tun_id=103,dl_dst=52:54:00:12:34:56,actions=output:10"
        ovs-ofctl add-flow br0 "table=1,tun_id=103,dl_dst=52:54:00:12:34:11,actions=output:1"
        ovs-ofctl add-flow br0 "table=1,tun_id=103,dl_type=0x0806,nw_dst=10.0.0.1,actions=output:1"
        ovs-ofctl add-flow br0 "table=1,tun_id=103,dl_type=0x0806,nw_dst=10.0.0.2,actions=output:10"

## vxlan

Q: What's a VXLAN?  
A: VXLAN stands for Virtual eXtensible Local Area Network, and is a means to solve the scaling challenges of VLAN networks in a multi-tenant environment.   VXLAN is an overlay network which transports an L2 network over an existing L3 network. For more information on VXLAN, please see RFC 7348:  
`http://tools.ietf.org/html/rfc7348`

Q: How much of the VXLAN protocol does Open vSwitch currently support?  
A: Open vSwitch currently supports the framing format for packets on the wire. `There is currently no support for the multicast aspects of VXLAN.  
To get around the lack of multicast support, it is possible to pre-provision MAC to IP address mappings either manually or from a controller.`

Q: What destination UDP port does the VXLAN implementation in Open vSwitch  
use?

A: By default, Open vSwitch will use the assigned IANA port for VXLAN, which is 4789. However, it is possible to configure the destination UDP port   manually on a per-VXLAN tunnel basis. An example of this configuration is provided below.  

    ovs-vsctl add-br br0
    ovs-vsctl add-port br0 vxlan1 -- set interface vxlan1 type=vxlan options:remote_ip=192.168.1.2 options:key=flow options:dst_port=8472


### Configuration example for vxlan

<img src="/images/other/Overlays-Blog-650x494.jpeg" alt="vxlan" width="1000"></a>

        sudo ovs-ofctl -O OpenFlow13  add-flow br0 " priority=0, actions=normal"
        sudo ovs-appctl fdb/show
        =========== For the Example, MAC Addresses are as follows ===========
        TEP1-192.168.1.180
        ------------------
         port  VLAN  MAC                Age
            2     0  00:00:00:00:00:01   11

        TEP2-192.168.1.181
        ------------------
         port  VLAN  MAC                Age
            2     0  00:00:00:00:00:05    3
            1     0  00:00:00:00:00:04    3

        TEP3-192.168.1.182
        ------------------
         port  VLAN  MAC                Age
            1     0  00:00:00:00:00:08   24


Host 192.168.1.180

    ovs-vsctl add-br br0
    ovs-vsctl set bridge br0 protocols=OpenFlow13
    ovs-vsctl add-port br0 vxlan -- set Interface vxlan type=vxlan options:key=flow options:local_ip=192.168.1.180 options:remote_ip=192.168.1.181 ofport_request=10
    ovs-vsctl add-port br0 vxlan1 -- set Interface vxlan1 type=vxlan options:key=flow options:local_ip=192.168.1.180 options:remote_ip=192.168.1.182 ofport_request=11

Add OpenFlow Flowmods using 3 tables, Classifier, Ingress, Egress. That is an implementation choice not a requirement.

    ovs-ofctl add-flow -O OpenFlow13 br0 "table=0,tun_id=0x5,in_port=10, actions=goto_table:2"
    ovs-ofctl add-flow -O OpenFlow13 br0 "table=0,tun_id=0x5,in_port=11 actions=goto_table:2"
    ovs-ofctl add-flow -O OpenFlow13 br0 "table=0,in_port=2,dl_src=00:00:00:00:00:01 actions=set_field:5->tun_id,goto_table=1"
    ovs-ofctl add-flow -O OpenFlow13 br0 "table=0,priority=16384,in_port=1 actions=drop"

    ovs-ofctl add-flow -O OpenFlow13 br0 "table=1,tun_id=0x5,dl_dst=00:00:00:00:00:08 actions=output:11,goto_table:2"
    ovs-ofctl add-flow -O OpenFlow13 br0 "table=1,tun_id=0x5,dl_dst=00:00:00:00:00:04 actions=output:10,goto_table:2"
    ovs-ofctl add-flow -O OpenFlow13 br0 "table=1,tun_id=0x5,dl_dst=00:00:00:00:00:05 actions=output:10,goto_table:2"
    ovs-ofctl add-flow -O OpenFlow13 br0 "table=1,priority=16384,tun_id=0x5,dl_dst=ff:ff:ff:ff:ff:ff actions=output:10,output:11,goto_table:2"
    ovs-ofctl add-flow -O OpenFlow13 br0 "table=1,priority=8192,tun_id=0x5 actions=goto_table:2"

    ovs-ofctl add-flow -O OpenFlow13 br0 "table=2,tun_id=0x5,dl_dst=00:00:00:00:00:01 actions=output:2"
    ovs-ofctl add-flow -O OpenFlow13 br0 "table=2,priority=16384,tun_id=0x5,dl_dst=ff:ff:ff:ff:ff:ff actions=output:2"
    ovs-ofctl add-flow -O OpenFlow13 br0 "table=2,priority=8192,tun_id=0x5 actions=drop"


Host 192.168.1.181

    ovs-vsctl add-br br0
    ovs-vsctl set bridge br0 protocols=OpenFlow13
    ovs-vsctl add-port br0 vxlan -- set Interface vxlan type=vxlan options:key=flow options:local_ip=192.168.1.181 options:remote_ip=192.168.1.182 ofport_request=10
    ovs-vsctl add-port br0 vxlan1 -- set Interface vxlan1 type=vxlan options:key=flow options:local_ip=192.168.1.181 options:remote_ip=192.168.1.180 ofport_request=11

Add OpenFlow Flowmods using 3 tables, Classifier, Ingress, Egress. That is an implementation choice not a requirement.

    ovs-ofctl add-flow -O OpenFlow13 br0 "table=0,tun_id=0x5,in_port=10 actions=goto_table:2"
    ovs-ofctl add-flow -O OpenFlow13 br0 "table=0,tun_id=0x5,in_port=11 actions=goto_table:2"

    ovs-ofctl add-flow -O OpenFlow13 br0 "table=0,in_port=1,dl_src=00:00:00:00:00:04 actions=set_field:5->tun_id,goto_table=1"
    ovs-ofctl add-flow -O OpenFlow13 br0 "table=0,in_port=2,dl_src=00:00:00:00:00:05 actions=set_field:5->tun_id,goto_table=1"
    ovs-ofctl add-flow -O OpenFlow13 br0 "table=0,priority=16384,in_port=1 actions=drop"
    ovs-ofctl add-flow -O OpenFlow13 br0 "table=0,priority=16384,in_port=2 actions=drop"

    ovs-ofctl add-flow -O OpenFlow13 br0 "table=1,tun_id=0x5,dl_dst=00:00:00:00:00:08 actions=output:10,goto_table:2"
    ovs-ofctl add-flow -O OpenFlow13 br0 "table=1,tun_id=0x5,dl_dst=00:00:00:00:00:02 actions=output:10,goto_table:2"
    ovs-ofctl add-flow -O OpenFlow13 br0 "table=1,tun_id=0x5,dl_dst=00:00:00:00:00:01 actions=output:11,goto_table:2"
    ovs-ofctl add-flow -O OpenFlow13 br0 "table=1,priority=16384,tun_id=0x5,dl_dst=ff:ff:ff:ff:ff:ff actions=output:10,output:11,goto_table:2"
    ovs-ofctl add-flow -O OpenFlow13 br0 "table=1,priority=8192,tun_id=0x5 actions=goto_table:2"

    ovs-ofctl add-flow -O OpenFlow13 br0 "table=2,tun_id=0x5,dl_dst=00:00:00:00:00:04 actions=output:1 VM1"
    ovs-ofctl add-flow -O OpenFlow13 br0 "table=2,tun_id=0x5,dl_dst=00:00:00:00:00:05 actions=output:2 VM2"
    ovs-ofctl add-flow -O OpenFlow13 br0 "table=2,priority=16384,tun_id=0x5,dl_dst=ff:ff:ff:ff:ff:ff actions=output:1,output:2"
    ovs-ofctl add-flow -O OpenFlow13 br0 "table=2,priority=8192,tun_id=0x5 actions=drop"

Host 192.168.1.182

    ovs-vsctl add-br br0
    ovs-vsctl set bridge br0 protocols=OpenFlow13
    ovs-vsctl add-port br0 vxlan -- set Interface vxlan type=vxlan options:key=10 options:local_ip=192.168.1.183 options:remote_ip=192.168.1.182  ofport_request=10
    ovs-vsctl add-port br0 vxlan -- set Interface vxlan type=vxlan options:key=10 options:local_ip=192.168.1.182 options:remote_ip=192.168.1.183 ofport_request=10

Add OpenFlow Flowmods using 3 tables, Classifier, Ingress, Egress. That is an implementation choice not a requirement.

    ovs-ofctl add-flow -O OpenFlow13 br0 "table=0,tun_id=0x5,in_port=10 actions=goto_table:2"
    ovs-ofctl add-flow -O OpenFlow13 br0 "table=0,tun_id=0x5,in_port=11 actions=goto_table:2"
    ovs-ofctl add-flow -O OpenFlow13 br0 "table=0,in_port=1,dl_src=00:00:00:00:00:08 actions=set_field:5->tun_id,goto_table=1,tun_dst:ip:1.1.1.1"
    ovs-ofctl add-flow -O OpenFlow13 br0 "table=0,priority=16384,in_port=1 actions=drop"

    ovs-ofctl add-flow -O OpenFlow13 br0 "table=1,tun_id=0x5,dl_dst=00:00:00:00:00:01 actions=output:11,goto_table:2"
    ovs-ofctl add-flow -O OpenFlow13 br0 "table=1,tun_id=0x5,dl_dst=00:00:00:00:00:02 actions=output:10,goto_table:2"
    ovs-ofctl add-flow -O OpenFlow13 br0 "table=1,tun_id=0x5,dl_dst=00:00:00:00:00:04 actions=output:10,goto_table:2"
    ovs-ofctl add-flow -O OpenFlow13 br0 "table=1,tun_id=0x5,dl_dst=00:00:00:00:00:05 actions=output:10,goto_table:2"
    ovs-ofctl add-flow -O OpenFlow13 br0 "table=1,priority=16384,tun_id=0x5,dl_dst=ff:ff:ff:ff:ff:ff actions=output:10,output:11,goto_table:2"
    ovs-ofctl add-flow -O OpenFlow13 br0 "table=1,priority=8192,tun_id=0x5 actions=goto_table:2"

    ovs-ofctl add-flow -O OpenFlow13 br0 "table=2,tun_id=0x5,dl_dst=00:00:00:00:00:08 actions=output:1"
    ovs-ofctl add-flow -O OpenFlow13 br0 "table=2,priority=16384,tun_id=0x5,dl_dst=ff:ff:ff:ff:ff:ff actions=output:1"
    ovs-ofctl add-flow -O OpenFlow13 br0 "table=2,priority=8192,tun_id=0x5 actions=drop"
