---
layout: post
title:  2016 SDN work plan
category: project
description:  List some planning SDN work in 2016
---

## ** introduction **
In order to speed SDN use in product, we will focus on the RCP/VNF work together with SDN.

i will use it to track the work.

## ** Dedicate task **


## ** web list **
 * [http://www.sdnlab.com/](http://www.sdnlab.com/)
 * [http://sdnia.org/](http://sdnia.org/)
 * [http://sdnhub.org/](http://sdnhub.org/)
 * []()
 * []()
 * []()
 * []()
 * []()


## ** reference list **



###  `Mobile SDN Architecture` by `Heikki Almay`

~~Telco Cloud SDN architecture~~

cloud stacks and orchestration software can use the SDN controller for configuring networks, for service, use a more scalable modular open approach.

  * separating the SDN controller and the application software
    + an instance of the basic SDN controller
    + on-top services
    + Open APIs allow innovating new services
 * controllers can be hierarchy Inside an administrative domain
 * Between administrative domains, information exchange between SDN controller is needed, BGP is one choice.

 inter-working of SDN with neighboring traditional IP/MPLS networks


~~Data center SDN~~  
SDN in the ETSI NFV framework:  
A data center SDN solution is typically integrated into the cloud stack. It provides the means to manage overlay networks across the virtualized infrastructure and to configure switches, routers and load balancers, firewalls etc. The network functions can be virtualized or appliance based.     
and it need integrat VNF managers and orchestration software to the SDN controller APIs.

<img src="/images/other/sdn-openstack.png" alt="sdn" width="1000"></a>

creating overlay networks on top of a traditional networking infrastructure

~~Backhaul SDN~~  
 * Nokia plans to integrate a virtual Cell Site Router (vCSR) in the base stations
 * SON for Mobile Backhaul


~~Front-haul SDN~~  

* between an eNB and Remote Radio Head (RRH)
* through optical fiber to the eNB
* One RRH can serve multiple cells

~~Evolution to SDN~~

  * Introducing NFV with SDN in a mobile network, to match the following requirement.
     + Management and orchestration needs to support both bare metal and virtualized network functions
     + all plane(user, control, manage) have their own IP address planning
     + Security relies more on traffic separation (beyond the  ~16 networks supported)
     + Protocols (GTP, SCTP，DIAMETER) are widely used
     + routing without NAT is needed

  * SDN as part of the Nokia Telco Cloud evolution
    + OpenStack is complemented by an OpenDaylight based SDN controller  
    + OpenFlow based front-hauling is an obvious  SDN use cases
    + backhaul forwarding box (OpenFlow forwarding and IP/MPLS interworking) be integrated into the base stations
