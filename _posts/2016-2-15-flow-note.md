---
layout: post
title:  traffic flow study note
category: blog
description:  traffic flow study note
---


## introduction
Here add some study note for LB.

##  what is flow

In packet switching networks, traffic flow, packet flow or network flow is a sequence of packets from a source computer to a destination, which may be another host, a multicast group, or a broadcast domain.

Another three fundamental options:

 * Frequency-division multiplexing (which includes radio and xWDM)
 * Time-division multiplexing (SONET, SDH, T1, E1…)
 * Statistical multiplexing (cells, frames, packets, datagrams…).

## packet-based

 Every single device operating at layers 2-4 does the same thing:

  * Receive a packet;
  * Inspect the packet header and extract the relevant fields;
  * Perform forwarding table (or TCAM) lookup to find the output port(s);
  * Send the packet to output queue(s).

packet-by-packet (stateless) forwarding in linux works like this:

  * Receive a packet;
  * Pass packet through ingress ACL (and whatever other input policy you have);
  * Perform a forwarding lookup (L2, L3, PBR, whatever);
  * Pass packet through egress ACL (and other output policies).

## Flow-based

cache-based forwarding mechanism that:  

  * Performs packet forwarding for the first packet in a flow;
  * Caches the results (output interface, rewriting, logging, counting and QoS actions);
  * Performs cache lookup on subsequent packets of the same flow and applies cached results without evaluating the input and output path.

good:  

  * It’s cheap enough to implement large flow caches that can easily cope with the maximum number of flows the device could reasonably have to handle;
  * The cost (in terms of hardware utilization or latency) of full pipeline processing significantly exceeds the cost of doing a cache lookup and applying the cached actions;
  * It’s possible to protect the device using flow-based forwarding against cache exhaustion, either by using a very large cache size or by flow-setup policing (example: TCP SYN cookies).  

## What Could OpenFlow Do?

  * match on any part of the packet header and select output port based on matech.
  * has actions to rewrite IP addresses and port numbers, it could implement NAT or PAT
  * no actions that would work beyond TCP/UDP port numbers.

##  Reactive or Proactive?  
The flows could be preset based on known network topology (= proactive flow setup) or created dynamically based on actual traffic (= reactive flow setup).  

In proactive flow setup, the controller creates the flows and stays away from the data plane. In reactive mode, the controller gets involved whenever a new flow needs to be created based on actual traffic load.

Note:  
session-based flows (that doesn’t work well even in virtual switches). You could use crude IP source+destination-based load balancing, and get the controller involved only when a different source IP address appears in the network

## OpenFlow-based load balancing ?
some factors which need consider:  

  * Amount of flow forwarding hardware
  * TCAM update speed  
    how much flows can be install per second ? around 1000 flows?
  * Hardware and OpenFlow implementation limit
  * Increased latency.  
    the reactive flow setup creates additional latency

some people comment:  
"OpenFlow-based load balancing makes sense only when you’re balancing a low number of high-bandwidth sessions toward an anycast server farm (so the load balancing switch doesn’t have to do address- or port rewrites)"    
