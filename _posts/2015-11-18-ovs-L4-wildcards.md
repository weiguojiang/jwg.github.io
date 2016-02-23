---
layout:     post
title:      ovs-L4port-wildcards
category: project
description: test steps for port wild-card
---


----test TCP port mask----

    $ sudo mn --arp --mac --topo=linear,2 --controller=remote,ip=10.56.212.56
    $ xterm h1 h1 h2 h2 s1
in xterm h1:

    # iperf -s -p 4112
    # iperf -s -p 4097
in xterm h2:(test connection with TCP succeed)

    # iperf -c 10.0.0.1 -p 4112
    # iperf -c 10.0.0.1 -p 4097
in xterm s1, add drop flow with mask 4112/0xff00

    # ovs-ofctl add-flow s1 dl_type=0x800,nw_proto=6,tcp_dst=4112/0xff00,actions=drop
in xterm h2:(verify the connection with TCP failed)

    # iperf -c 10.0.0.1 -p 4112
    # iperf -c 10.0.0.1 -p 4097
in xterm s1, remove drop flow with mask 4112/0xff00

    # ovs-ofctl del-flows s1 dl_type=0x800,nw_proto=6,tcp_dst=4112/0xff00
in xterm h2:(test again, the connection works again)
    # iperf -c 10.0.0.1 -p 4112
    # iperf -c 10.0.0.1 -p 4097

NOTE: the mask could be 4097/0xff00 0x10xx/0xff00

----test UDP port mask----

    $ sudo mn --arp --mac --topo=linear,2 --controller=remote,ip=10.56.212.56
    $ xterm h1 h1 h2 h2 s1
in xterm h1:
    # nc -ul -p 4112
    # nc -ul -p 4097
in xterm h2:(test connection with TCP succeed)

    # nc -u 10.0.0.1 4112
    # nc -u 10.0.0.1 4097
in xterm s1, add drop flow with mask 4112/0xff00

    # ovs-ofctl add-flow s1 dl_type=0x800,nw_proto=17,udp_dst=4112/0xff00,actions=drop
in xterm h2:(verify the connection with TCP failed)

    # nc -u 10.0.0.1 4112
    # nc -u 10.0.0.1 4097
in xterm s1, remove drop flow with mask 4112/0xff00

    # ovs-ofctl del-flows s1 dl_type=0x800,nw_proto=17,udp_dst=4112/0xff00

in xterm h2:(test again, the connection works again)

    # nc -u 10.0.0.1 4112
    # nc -u 10.0.0.1 4097

----example in OVS server----

The user space flow list below:

    root@odl1:/home/odl1# ovs-ofctl add-flow ovs dl_type=0x0800,nw_proto=17,udp_dst=2345/0xfff0,actions=drop
    root@odl1:/home/odl1# ovs-ofctl dump-flows ovs
    NXST_FLOW reply (xid=0x4):
    cookie=0x0, duration=1847.564s, table=0, n_packets=0, n_bytes=0, idle_age=1847, priority=33,vlan_tci=0x0000/0x1fff,dl_src=00:1f:29:c5:d1:34 actions=mod_vlan_vid:2000,output:1
    cookie=0x2b0000000000000b, duration=1947.255s, table=0, n_packets=3, n_bytes=534, idle_age=1944, priority=0 actions=drop
    cookie=0x0, duration=2.908s, table=0, n_packets=0, n_bytes=0, idle_age=2, udp,tp_dst=0x920/0xfff0 actions=drop
    cookie=0x2b00000000000010, duration=1843.689s, table=0, n_packets=1266, n_bytes=94482, idle_age=0, priority=2,in_port=1 actions=output:2,CONTROLLER:65535
    cookie=0x2b0000000000000f, duration=1843.689s, table=0, n_packets=1, n_bytes=60, idle_age=1893, priority=2,in_port=2 actions=output:1,CONTROLLER:65535
    cookie=0x0, duration=1847.462s, table=0, n_packets=0, n_bytes=0, idle_age=1847, priority=22,dl_vlan=2000,dl_dst=00:1f:29:c5:d1:34 actions=strip_vlan,output:2
    cookie=0x2b0000000000000b, duration=1947.255s, table=0, n_packets=0, n_bytes=0, idle_age=1947, priority=100,dl_type=0x88cc actions=CONTROLLER:65535
    root@odl1:/home/odl1#

The kernel space flows list below. Since the DP's flows are installed only in the runtime, here use the TCP _ iperf _ testing. Note, the dst port will appear 4097 or 4112 in the runtime, but at the same period, only one will be installed in DP.

    root@odl-VirtualBox:~# ovs-dpctl dump-flows
    recirc_id(0),skb_priority(0),in_port(5),eth_type(0x0800),ipv4(src=10.0.0.2/0.0.0.0,dst=10.0.0.1/0.0.0.0,proto=6/0xff,tos=0/0,ttl=64/0,frag=no/0xff),tcp(src=35229/0,dst=4097/0xff00), packets:0, bytes:0, used:never, actions:drop
    recirc_id(0),skb_priority(0),in_port(1),eth_type(0x0800),ipv4(src=10.0.0.2/0.0.0.0,dst=10.0.0.1/0.0.0.0,proto=6/0,tos=0/0,ttl=64/0,frag=no/0xff), packets:0, bytes:0, used:never, actions:userspace(pid=4294958098,slow_path(controller))
    root@odl-VirtualBox:~#
    root@odl-VirtualBox:~# ovs-dpctl dump-flows
    recirc_id(0),skb_priority(0),in_port(2),eth_type(0x0800),ipv4(src=0.0.0.0/0.0.0.0,dst=255.255.255.255/0.0.0.0,proto=17/0,tos=0x10/0,ttl=128/0,frag=no/0xff), packets:0, bytes:0, used:never, actions:userspace(pid=4294958097,slow_path(controller))
    recirc_id(0),skb_priority(0),in_port(5),eth_type(0x0800),ipv4(src=10.0.0.2/0.0.0.0,dst=10.0.0.1/0.0.0.0,proto=6/0xff,tos=0/0,ttl=64/0,frag=no/0xff),tcp(src=40841/0,dst=4112/0xff00), packets:0, bytes:0, used:never, actions:drop
    recirc_id(0),skb_priority(0),in_port(1),eth_type(0x0800),ipv4(src=10.0.0.2/0.0.0.0,dst=10.0.0.1/0.0.0.0,proto=6/0,tos=0/0,ttl=64/0,frag=no/0xff), packets:0, bytes:0, used:never, actions:userspace(pid=4294958098,slow_path(controller))
    root@odl-VirtualBox:~#
