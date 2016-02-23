---
layout: post
title:  fronthual SDN autoconfiguration
category: blog
description:  analyze network auto-configuration in fronthual env
---

## refer link:


## Requirement
in FrontHaul env, when the FZM start, it will send the DHCP packet to DHCP server(locates in the VM),
the ToR switch and access switch need do configuration to let the packet travel to vm.

## Technical analyze
Seems ODL doesn't trigger Topo change after it receives the DHCP packet based on the test result.
so we need check how ODL handle the DHCP packet firstly.

### DPCP background

you can get more information from wikipedia via  [wikipedia](https://en.wikipedia.org/wiki/Dynamic_Host_Configuration_Protocol)

 * DHCP discovery
 * DHCP offer
 * DHCP request
 * DHCP ack

The DHCP operation begins with clients broadcasting a request.
The client broadcasts messages on the network subnet using the destination address 255.255.255.255 or the specific subnet broadcast address.

### how ODL process DHCP

From our test, seems now Controller just broadcast the packet as soon as it receives the DHCP
packet

### how ODL process General Broadcast Traffic

 * broadcast drop: drops all non-ARP/DHCP broadcast traffic.
 * broadcast forward-to-known: forwards broadcast traffic to all known hosts
 * broadcast always-flood: always flood the broadcast traffic.

### possible solutions

#### `Controller can act as a DHCP server`
 http://events.linuxfoundation.org/sites/events/files/slides/DHCPServiceinODL.pdf

 If controller is acting as a backend network virtualization solution
 (running ovsdb,vtn,dove) this option is better. Because in this setup, virtualization services knows the mac to ip mapping and they can resolve the DHCP request, But it require the implementation of full dhcp server capability in the controller that can work with existing virtualization solutions in ODL.

#### `internal APP to trigger Topo change`
we can design one internal module with the following behavior:

 * after it receive the first DHCP req, it store the host/mac/port/dpid information to databroke.
	- so externl APP can register this data change to know the FZM information.
 * user can do BTS configuration by RestAPI and APP need do configuration to SDN network.
	- do flow create/delete.

### possible used scenario
now in CloudBTS, it intend to use the solution which small cell has used for FZM auto-discovery.
here is the link:

 * https://sharenet-ims.int.net.nokia.com/Overview/D526717584

we need study it whether SDN can cover this scenario.
