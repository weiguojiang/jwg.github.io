---
layout: post
title:  fabric study note
category: blog
description:  fabric study note
---


## introduction
it focus on the data center clos network.

## scenarios

 * layer-3 fabrics
 * layer-2 fabrics
 * mixed layer-2-3 fabrics

## consider factors

  * non-redundant
  * redundant server-to-network connectivity
  * routing on servers and connectivity to external world via routed links or network services

### layer-3 fabrics
connectivity part is split into MLAG- and active/standby

Design considerations:  

   * routing protocol selection
   * convergence speed optimization
   * addressing on core links
   * route summarization
   * ECMP or LAG
   * BGP AS number selection
   * fabric deployments, configuration and troubleshooting ?

Host routing (propagating host routes across layer-3-only fabric) and routing on hosts.   
Design considerations:  

   * forwarding table sizes
   * routing protocol selection

### layer-2 fabrics
Layer-2 fabrics, including MLAG-based solutions, routing-on-layer-2 solutions (TRILL, SPB) and overlay fabrics (VXLAN)

## external connectivity and load balancing
