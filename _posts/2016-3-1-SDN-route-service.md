---
layout: post
title:  route as service
category: blog
description: route as service in SDN study
---


## introduction
In cloud, whether we need the route daemon to run OSPF/BGP for guest
is not known. can we just create default gateway for the guest application IP in cloud env ?
if cloud infrastructure can provide route as service, then it can.
in this doc, i will check the possibility.

## Reference link

[Brocade SDN Controller User Guide](https://www.brocade.com/content/html/en/user-guide/SDN-Controller-2.1.0-User-Guide/GUID-B1355D5E-A712-4F52-AFBD-EBBB530CA11E.html)  
[Juniper route use case](http://www.juniper.net/assets/us/en/local/pdf/whitepapers/2000594-en.pdf)  
[BGP-LS](http://www.slideshare.net/GilesHeron/bgpcep-odl-summit-2015?qid=f4f292e5-33f4-4052-a069-85a7e6f1d220&v=&b=&from_search=1)  
[network guide](https://www.rdoproject.org/networking/networking-in-too-much-detail/)  
[openstack network guide](http://docs.openstack.org/liberty/networking-guide/scenario_l3ha_ovs.html)  

[don't run ospf with your host server](http://blog.ipspace.net/2016/03/dont-run-ospf-with-your-customers.html)

## run ospf or bgp in your host server ?

### Technical Differences

OSPF is an Interior Routing Protocol designed to exchange information within an autonomous system.
BGP is an Exterior Routing Protocol with enough safeguards to be used between autonomous systems.

BGP uses a pretty conservative approach to information propagation: receive -> filter -> evaluate -> filter -> propagate best information.
OSPF is focused on speed-of-convergence and uses a radically different approach: receive -> flood everything -> evaluate.

In other words, anyone who’s part of an OSPF domain can insert any stupidity they wish into the domain and there’s nothing anyone else can do to stop the propagation of that stupidity within an area, and it stays in the area for at least half an hour. There are (as expected) vendor-specific kludges one can use between areas, but within area flooding rules (and external routes get flooded across area boundaries unless you use NSSA areas).

### analysis

Routing protocols running on virtual appliances significantly increase the flexibility of virtual-to-physical network integration – you can easily move the whole application stack across subnets or data centers without changing the physical network configuration.

Don’t use link-state routing protocols

Link-state routing protocols rely on shared topology database flooded between participating nodes (routers). The whole link state domain is a single trust zone – a single node going bonkers can bring down the whole domain.

Conclusion: don’t use link-state routing protocols between mission-critical physical network infrastructure and virtual appliances. BGP is the only safe choice.

## unknown questions

 * can SDN network exchange the route information as one AS with other AS and how achieve it?

## Openstack network connectivity

### Scenario: classic with Open vSwitch
<img src="/images/other/Neutron_architecture.png" alt="sdn" width="1000"></a>

Bridge device exists to support firewall rules. you can check it by  
        `# iptables -S | grep tap`

### Scenario: Provider networks with Open vSwitch

<img src="/images/other/scenario-provider-ovs-flowns1.png" alt="sdn" width="1000"></a>

---
<img src="/images/other/scenario-provider-ovs-flowew2.png" alt="sdn" width="1000"></a>

### Scenario: DVR with Open vSwitch

<img src="/images/other/scenario-dvr-general.png" alt="sdn" width="1000"></a>

---

<img src="/images/other/scenario-dvr-flowns2.png" alt="sdn" width="1000"></a>

---
