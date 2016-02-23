---
layout: post
title: setVxLANid-in-OVS-flow
category: project
description: test steps for set VxLAN VNI in OVS flow
---

## ** refer link **
 * http://10.140.88.123/vxlan/

## Setup env

<img src="/images/other/setVxLANid_in_OVS_flow.jpg" width="1000"></a>

ODL: Lithium SR1

## OVS1 configuration

    ovs-vsctl del-br br0
    ovs-vsctl add-br br0
    ovs-vsctl add-port br0 eth3
    ovs-vsctl add-port br0 tap0
    ifconfig eth1 192.169.0.11/24 up
    ovs-vsctl add-port br0 vx0 -- set interface vx0 type=vxlan options:remote_ip=192.169.0.33 option:key=flow ofport_request=13
    ovs-vsctl set-controller br0 tcp:10.56.212.53:6633

    ovs-ofctl add-flow br0 "table=0,hard_timeout=0,idle_timeout=0,priority=33,dl_type=0x800,nw_proto=17,udp_dst=0x300/0xff00,actions=set_field:13->tun_id,resubmit(,1)"
    ovs-ofctl add-flow br0 "table=1,hard_timeout=0,idle_timeout=0,priority=33,dl_type=0x800,nw_proto=17,udp_dst=0x300/0xff00,actions=output:13"
    ovs-ofctl add-flow br0 "table=0,hard_timeout=0,idle_timeout=0,priority=33,dl_type=0x800,nw_proto=17,udp_dst=0x100/0xff00,actions=output:2"

## OVS3 configuration

    ovs-vsctl del-br br0
    ovs-vsctl add-br br0
    ovs-vsctl add-port br0 eth3
    ovs-vsctl add-port br0 tap0
    ifconfig eth1 192.169.0.33/24 up
    ovs-vsctl add-port br0 vx0 -- set interface vx0 type=vxlan options:remote_ip=192.169.0.11 option:key=flow ofport_request=13
    ovs-vsctl set-controller br0 tcp:10.56.212.53:6633

    ovs-ofctl add-flow br0 "table=0,hard_timeout=0,idle_timeout=0,priority=33,dl_type=0x800,nw_proto=17,udp_dst=0x300/0xff00,actions=output:2"
    ovs-ofctl add-flow br0 "table=0,hard_timeout=0,idle_timeout=0,priority=33,dl_type=0x800,nw_proto=17,udp_dst=0x100/0xff00,actions=set_field:13->tun_id,resubmit(,1)"
    ovs-ofctl add-flow br0 "table=1,hard_timeout=0,idle_timeout=0,priority=33,dl_type=0x800,nw_proto=17,udp_dst=0x100/0xff00,actions=output:13"

## Testing output

    root@odl1:~# ovs-ofctl dump-flows br0
    NXST_FLOW reply (xid=0x4):
     cookie=0x0, duration=4228.748s, table=0, n_packets=16, n_bytes=964, idle_age=149, priority=33,udp,tp_dst=0x300/0xff00 actions=load:0xd->NXM_NX_TUN_ID[],resubmit(,1)
     cookie=0x0, duration=4199.842s, table=0, n_packets=1, n_bytes=60, idle_age=2696, priority=33,udp,tp_dst=0x100/0xff00 actions=output:2
     cookie=0x2b00000000000123, duration=4271.475s, table=0, n_packets=3, n_bytes=180, idle_age=4267, priority=0 actions=drop
     cookie=0x2b00000000000388, duration=4122.898s, table=0, n_packets=447, n_bytes=42324, idle_age=3, priority=2,in_port=13 actions=output:2,output:1,CONTROLLER:65535
     cookie=0x2b00000000000389, duration=4122.898s, table=0, n_packets=3011, n_bytes=210127, idle_age=1, priority=2,in_port=1 actions=output:2,output:13
     cookie=0x2b00000000000387, duration=4122.898s, table=0, n_packets=22, n_bytes=1635, idle_age=144, priority=2,in_port=2 actions=output:13,output:1,CONTROLLER:65535
     cookie=0x2b00000000000123, duration=4271.475s, table=0, n_packets=1680, n_bytes=187306, idle_age=3, priority=100,dl_type=0x88cc actions=CONTROLLER:65535
     cookie=0x0, duration=4207.824s, table=1, n_packets=16, n_bytes=964, idle_age=149, priority=33,udp,tp_dst=0x300/0xff00 actions=output:13


    root@odl3:/home/odl3# ovs-ofctl dump-flows br0
    NXST_FLOW reply (xid=0x4):
     cookie=0x0, duration=4223.393s, table=0, n_packets=16, n_bytes=964, idle_age=253, priority=33,udp,tp_dst=0x300/0xff00 actions=output:2
     cookie=0x0, duration=4214.484s, table=0, n_packets=0, n_bytes=0, idle_age=4214, priority=33,udp,tp_dst=0x100/0xff00 actions=load:0xd->NXM_NX_TUN_ID[],resubmit(,1)
     cookie=0x2b00000000000125, duration=4231.097s, table=0, n_packets=2, n_bytes=120, idle_age=4227, priority=0 actions=drop
     cookie=0x2b00000000000382, duration=4227.097s, table=0, n_packets=3007, n_bytes=209833, idle_age=1, priority=2,in_port=13 actions=output:2,output:1,CONTROLLER:65535
     cookie=0x2b00000000000383, duration=4227.097s, table=0, n_packets=410, n_bytes=40041, idle_age=5, priority=2,in_port=1 actions=output:2,output:13,CONTROLLER:65535
     cookie=0x2b00000000000381, duration=4227.097s, table=0, n_packets=56, n_bytes=3882, idle_age=248, priority=2,in_port=2 actions=output:13,output:1,CONTROLLER:65535
     cookie=0x2b00000000000125, duration=4231.097s, table=0, n_packets=846, n_bytes=94752, idle_age=2, priority=100,dl_type=0x88cc actions=CONTROLLER:65535
     cookie=0x0, duration=4206.816s, table=1, n_packets=0, n_bytes=0, idle_age=4206, priority=33,udp,tp_dst=0x100/0xff00 actions=output:13
    root@odl3:/home/odl3#

## wireshark packets

<a href="/data/SetVxLANid_in_OVS_flow.pcapng">download</a>
