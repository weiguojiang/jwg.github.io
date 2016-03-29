---
layout: post
title:  general load balance knowledge
category: blog
description: general load balance knowledge
---

## general introduction

### what is Load Balance ?

负载设备作为服务器的前端设备，负责将流量根据各种策略分发到后端各个真正的服务器上，从而使后端的服务器群得到合理有效的应用

### why we need Load Balance

保证网络的可扩展性，服务器的易管理性和可靠性

由于目前现有网络的各个核心部分随着业务量的提高，访问量和数据流量的快速增长，其处理能力和计算强度也相应地增大，使得单一的服务器设备根本无法承担。在此情况下，如果扔掉现有设备去做大量的硬件升级，这样将造成现有资源的浪费，而且如果再面临下一次业务量的提升时，这又将导致再一次硬件升级的高额成本投入，甚至性能再卓越的设备也不能满足当前业务量增长的需求.

将负载（工作任务）进行平衡、分摊到多个操作单元上进行执行，例如Web服务器、FTP服务器、企业关键应用服务器和其它关键任务服务器等，从而共同完成工作任务

负载均衡建立在现有网络结构之上，它提供了一种廉价又有效的方法扩展网络设备和服务器的带宽、 增加吞吐量、加强网络数据处理能力、提高网络的灵活性和可用性

### how it work

在第一个包的时候，根据策略选择后端的服务器，建立session，也就是slow path。后面的包通过session表，得到session后，直接利用上次的结果，将包发给相同的服务器，也就是fast path。

负载均衡有两方面的含义：首先，大量的并发访问或数据流量分担到多台节点设备上分别处理，减少用户等待响应的时间；其次，单个重负载的运算分担到多台节点设备上做并行处理，每个节点设备处理结束后，将结果汇总，返回给用户，系统处理能力得到大幅度提高

通常，负载 均衡会根据网络的不同层次（网络七层）来划分。其中，第二层的负载均衡指将多条物理链路当作一条单一的聚合逻辑链路使用，这就是链路聚合 （Trunking）技术，它不是一种独立的设备，而是交换机等网络设备的常用技术。

现代负载均衡技术通常操作于网络的第四层或第七层，这是针对网络应用 的负载均衡技术，它完全脱离于交换机、服务器而成为独立的技术设备。这也是我们现在要讨论的对象。

### how it select

* 依序Round Robin 　
* 比重Weighted Round Robin
* 流量比例Traffic
* 使用者端User 　
* 应用别Application
* 联机数量Session
* 服务别Service
* 自动分配Auto Mode

### deployment

负载均衡有三种部署方式：路由模式、桥接模式、服务直接返回模式。

路由模式
服务器的网关必须设置成负载均衡机的LAN口地址，且与WAN口分署不同的逻辑网络。因此所有返回的流量也都经过负载均衡。这种方式对网络的改动小，能均衡任何下行流量。

桥接模式
桥接模式配置简单，不改变现有网络。负载均衡的WAN口和LAN口分别连接上行设备和下行服务器。LAN口不需要配置IP（WAN口与LAN口是桥连接），所有的服务器与负载均衡均在同一逻辑网络中

服务直接返回模式
这种安装方式负载均衡的LAN口不使用，WAN口与服务器在同一个网络中，互联网的客户端访问负载均衡的虚IP（VIP），虚IP对应负载均衡机的WAN口，
载均衡根据策略将流量分发到服务器上，服务器直接响应客户端的请求。因此对于客户端而言，响应他的IP不是负载均衡机的虚IP（VIP），而是服务器自身的IP地址。也就是说返回的流量是不经过负载均衡的。因此这种方式适用大流量高带宽要求的服务。


## Ananta: Cloud Scale Load Balancing

contributions:  

 * Identifying the requirements and design space for a cloudscale solution for layer-4 load balancing.
 * Providing design, implementation and evaluation of Ananta that combines techniques in networking and distributed systems
   to refactor load balancer functionality in a novel way to meet scale, performance and reliability requirements.
 * Providing measurements and insights from running Anant in a large operational Cloud.

Ananta, a scale-out layer-4 load balancer that runs on commodity hardware and
meets the performance, reliability and operational requirements of multi-tenant cloud computing environments

Ananta combines existing techniques in routing and distributed systems in a unique way and
splits the components of a load balancer into a consensus-based reliable control plane and a decentralized scale-out data plane

an agent in every host that can take over the packet modification function.
direct server return (DSR) and network address translation (NAT) capabilities across layer-2 boundaries.

<img src="/images/LB/Ananta/data-plane-tiers.png" alt="sdn" width="1000"></a>

at the topmost/second/third tiers:

  * routers provide load distribution at the network layer (layer-3) based on the Equal Cost Multi Path (ECMP) routing protocol
  * a scalable set of dedicated servers for load balancing, called multiplexers (Mux), maintain connection flow state in memory
  and do layer-4 load distribution to application servers
  * virtual switch on every server provides stateful NAT functionality.

no outbound trafffic has to pass through the Mux  and ability to offload multiplexer functionality down to the host.

The SDN controller

maintains high availability via state replication based on the Paxos [14] distributed consensus protocol.
The controller also implements real-time port allocation for outbound NAT, also known as Source NAT (SNAT)

keeping per-connection state is necessary to maintain application uptime due to the dynamic nature of the cloud.
weighted random load balancing policy, which reduces the need for per-flow state synchronization among load balancer instances.


Background:

 * Each machine is assigned a private Direct IP (DIP) address
 * service is assigned one public Virtual IP (VIP) address
 * A service exposes zero or more external endpoints that each receive inbound traffic on a specific protocol and port on the VIP
 * All outbound traffic originating from a service is Source NAT’ed (SNAT) using the same VIP address as well

Design：

 * assumptions： load balancing policies that require global knowledge，randomly distributing connections across servers
based on their weights (The weights are derived based on the size of the VM or other capacity metrics)
 * offloads significant data plane and control plane functionality down to the hypervisor in end systems. The hypervisor
needs to handle state only for the VMs hosted on it.

### Architecture:

Ananta is a loosely coupled distributed system comprising three main components:  

 * Ananta Manager (AM),
 * Multiplexer (Mux) and
 * Host Agent (HA).

<img src="/images/LB/Ananta/Arch.png" alt="sdn" width="1000"></a>

we first discuss the load balancer configuration and the overall packet flow

<img src="/images/LB/Ananta/in-conn.png" alt="sdn" width="1000"></a>

 * routers to distribute packets destined for the VIP across all
the Mux nodes based on ECMP
 * the Mux chooses a DIP for the connection based on its load balancing algorithm, It then encapsulates the received packet using IP-in-IP protocol
  setting the selected DIP as the destination address in the outer header
 * sends it out using regular IP routing at the Mux
 * The HA, located on the same physical machine as the target DIP, intercepts this encapsulated packet,
   removes the outer header, and rewrites the destination address and port.
 * remembers this NAT state. The HA then sends the rewritten packet to the VM
 * When the VM sends a reply packet for this connection, it is intercepted by the HA
 * The HA does a reverse NAT based on the state and rewrites the source address and port
    It then sends the packet out to the router towards the source of this connection

Note:  
Not all packets of a connection would end up at the same Mux, however all packets for a single
connection must be delivered to the same DIP.   
Muxes achieve this via a combination of consistent hashing and state management

<img src="/images/LB/Ananta/out-conn.png" alt="sdn" width="1000"></a>

 * A VM sends a packet containing its DIP as the source address, portd as the port and an external address as
the destination address.
 * The HA intercepts this packet and recognizes that this packet needs SNAT. It then holds the packet
in a queue and sends a message to AM requesting an externally routable VIP and a port for this connection
 * AM allocates a (V IP; ports) from a pool of available ports and configures each Mux with this allocation.
 * AM then sends this allocation to the HA.
 * The HA uses this allocation to rewrite the packet so that its source address and port are now (V IP; ports).
   The HA sends this rewritten packet directly to the router. The return packets from the external destination are handled similar to inbound connections.
   The return packet is sent by the router to one of the Mux nodes.
 * The Mux already knows that DIP2 should receive this packet , so it encapsulates the packet with DIP2 as the destination and sends it out
 * The HA intercepts the return packet, performs a reverse translation so that the packet’s destination address and port are now
   (DIP; portd). The HA sends this packet to the VM.

   <img src="/images/LB/Ananta/fast-conn.png" alt="sdn" width="1000"></a>


### Mux

Route Management  
Each Mux is a BGP speaker. It starts advertising a route for that VIP to its firsthop router with itself as the next hop.
Running the BGP protocol on the Mux provides automatic failure detection and recovery.

Packet Handling  

a mapping table (VIP map )  

  * a VIP endpoint, i.e., three-tuple (VIP, IP protocol, port), to a list of DIPs
  * is computed by AM and sent to all the Muxes in a Mux Pool

it computes a hash and  uses this hash to lookup a DIP from the list of DIPs in the associated map.
it encapsulates packet with an outer IP header and forwards this packet to the DIP.

Note:  
All Muxes in a Mux Pool use the exact same hash function and seed value.
Since all Muxes have the same mapping table,
it doesn’t matter which Mux a given new connection goes to, it will be directed to the same DIP

Stateful entries are used for load balancing and stateless entries are used for SNAT.
Every non-SYN TCP packet, and every packet for connection-less protocols, is matched against this flow table first,
and if a match is found it is forwarded to the DIP from the flow table. This ensures that once a connection is directed to a DIP


## Maglev: A Fast and Reliable Software Network Load Balancer

contributions are:   
1) present the design and implementation of Maglev,   
2) share experiences of operating Maglev at a global scale,   
3) demonstrate the capability of Maglev through extensive evaluations.

<img src="/images/LB/Maglev/gen.png" alt="sdn" width="1000"></a>

Maglev is Google’s network load balancer. It is a large distributed software system that runs on commodity Linux servers.

Network routers distribute packets evenly to the Maglev machines via Equal Cost Multipath (ECMP);
each Maglev machine then matches the packets to their corresponding services and spreads them evenly to the service endpoints.

Maglev is optimized for packet processing performance and be also equipped with consistent hashing and connection tracking features.

the design and implementation are highly complex and challenging:  

 * each individual machine in the system must provide high throughput.
 * provide connection persistence: packets belonging to the same connection should always be directed to the same service endpoint

### System Overview

<img src="/images/LB/Maglev/packet-flow.png" alt="sdn" width="1000"></a>

Every Google service has one or more Virtual IP ad-dresses (VIPs). A VIP served by `multiple service endpoints` behind Maglev.
Maglev associates each VIP with a set of service endpoints and announces it to the router over BGP.

The setup for large clusters is more complicated to be scaling, so hardware encapsulators are deployed behind
the router, which tunnel packets from routers to Maglev machines.

#### config

<img src="/images/LB/Maglev/config.png" alt="sdn" width="1000"></a>

each Maglev machine contains a controller and a forwarder. it earn the VIPs to be served from configuration objects.
the controller periodically checks the health status of the forwarder and decides whether to announce or withdraw all the VIPs via BGP.

At the forwarder, each VIP is configured with one or more backend pools.
Each backend pool is associated with one or more health checking methods by IP.

### Forwarder

<img src="/images/LB/Maglev/forward.png" alt="sdn" width="1000"></a>

The forwarder receives packets from the NIC, rewrites them with proper GRE/IP headers and then sends them back to the NIC.

After Packets received by the NIC, steering module process it, it calculates the `5-tuple hash1`  and assigns them to different `receiving queues` based on hash value(Each receiving queue is attached to a packet rewriter thread).

The packet thread first tries to match each packet to a `configured VIP`. Then it recomputes the `5-tuple hash` and
looks up the hash value in the `connection tracking table`. (The connection table stores backend selection results for recent connections)

If a match is found and the selected backend is still healthy, the result is simply reused.
Otherwise the thread consults the `consistent hashing module` and selects a new backend;
it also adds an entry to the connection table for future packets with the same 5-tuple.

The forwarder maintains one connection table per packet thread to avoid access contention.

After a backend is selected, the packet thread encapsulates the packetwith proper GRE/IP headers
and sends it to the attached transmission queue. The muxing module then polls all transmission queues and
passes the packets to the NIC.

#### Fast Packet Processing

<img src="/images/LB/Maglev/fast.png" alt="sdn" width="1000"></a>

Maglev bypass the kernel entirely for packet processing.

When Maglev is started, it pre-allocates a packet pool that is shared between the NIC and the forwarder.
Both the steering and muxing modules maintain a ring queue of pointers pointing to packets in the packet pool.

### backend selection

it need send all packets of a connection to the same backend:  

  * First, we select a backend using a new form of consistent hashing which distributes traffic very evenly.
  * Then we record the selection in a local connection tracking table.

connection tracking table:  
 uses a fixed-size hash table mapping 5-tuple hash values to backends.

### Consistent Hashing

how about share connection state among all Maglev machines ?  
  it may negatively affect forwarding performance

A better-performing solution is to use local consistent hashing, The idea is to generate a large lookup table with each
backend taking a number of entries in the table  

 * load balancing: each backend will receive an almost equal number of connections.
 * minimal disruption: when the set of backends changes, a connectionwill likely be sent to the same backend as it was before.

 Maglev hashing


### Fragment Handling

assumptions:

* First, all fragments of the same datagram must be received by the same Maglev.
* Second, theMaglev must make consistent backend selection decisions for unfragmented packets, first fragments, and non-first fragments.

but some routers use 5-tuple hashing for first fragments and 3-tuple for non-first fragments.  
so Each Maglev is configured with a special backend pool consisting of all Maglevs within the cluster.
Upon receipt of a fragment, Maglev computes its 3-tuple hash using the L3 header and forwards it to a Maglev from the pool based on thehash value.  
Since all fragments belonging to the same datagram contain the same 3-tuple, they are guaranteed to be redirected to the same Maglev. We use the GRE
recursion control field to ensure that fragments are only redirected once


## Reference link

[network guide](https://www.rdoproject.org/networking/networking-in-too-much-detail/)
