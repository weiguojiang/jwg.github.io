---
layout: post
title:  ospf stud
category: blog
description:  ospf test configuration
---

## ** introduction **
here i will cover one test env and configuration.

<img src="/images/other/ospf.png" alt="sdn" width="1000"></a>

## ** env description **

here i create area 1 , area 0 and area 2, in area2, esw5 has one default route and it will distribute to ospf area.
this case is verified with GNS3 with 1.4 version.
all the router has creeate one IP addr on loopback interface and will be the root id.

1) DR and BDR select   
assume esw1 and esw2 startup with esw3 is still down
after esw1 startup, it will send one hello packet and then esw2 send hello packet. in the second hello packet, it will include the active neighbor. after both router is in two-way state, it start one timer to select the DR and BDR. during these time, it monitor the received hello packet, when it timerover, esw1 select esw2 as DR and it will fill it in the hello packet and send it.
and it will enter ex-start state. and two routers go full state.

then esw3 startup, it will send hello packet. it has the high root id
with the same priority, but esw1 has been DR, so esw3 is still DRother.

## ** Questions **
1) in Ethernet, hello packet is always sent with 224.0.0.5, is it correct ?   
   one strange thing is that i see hello is send to one specific IP.e.g. 2.2.2.2.


2) when update, DRother router send it to 224.0.0.6 and DR will send LSU with 224.0.0.5, all other router need ack it. does it ack to 224.0.0.6 or DR IP?

## wireshark packets

<a href="/data/ospf-startup.pcapng">download</a>

## ** router info **

      ESW1#show ip route ospf
      10.0.0.0/24 is subnetted, 3 subnets
      O IA    10.10.0.0 [110/2] via 10.10.1.3, 00:00:41, FastEthernet0/1
      O IA    10.10.2.0 [110/3] via 10.10.1.3, 00:00:41, FastEthernet0/1
      O*E2 0.0.0.0/0 [110/1] via 10.10.1.3, 00:00:41, FastEthernet0/1

      ESW1#show ip ospf neighbor

      Neighbor ID     Pri   State           Dead Time   Address         Interface
      2.2.2.2           1   FULL/BDR        00:00:34    10.10.1.2       FastEthernet0/1
      3.3.3.3           1   FULL/DR         00:00:30    10.10.1.3       FastEthernet0/1
      ESW1#show ip ospf database

                  OSPF Router with ID (1.1.1.1) (Process ID 1)

                      Router Link States (Area 1)

      Link ID         ADV Router      Age         Seq#       Checksum Link count
      1.1.1.1         1.1.1.1         61          0x80000003 0x005B9F 1
      2.2.2.2         2.2.2.2         61          0x80000003 0x001DD4 1
      3.3.3.3         3.3.3.3         62          0x80000002 0x00E305 1

                      Net Link States (Area 1)

      Link ID         ADV Router      Age         Seq#       Checksum
      10.10.1.3       3.3.3.3         62          0x80000001 0x0046B3

                      Summary Net Link States (Area 1)

      Link ID         ADV Router      Age         Seq#       Checksum
      10.10.0.0       3.3.3.3         102         0x80000001 0x003EDD
      10.10.2.0       3.3.3.3         58          0x80000001 0x0032E6

                      Summary ASB Link States (Area 1)

      Link ID         ADV Router      Age         Seq#       Checksum
      5.5.5.5         3.3.3.3         58          0x80000001 0x004ECB

                      Type-5 AS External Link States

      Link ID         ADV Router      Age         Seq#       Checksum Tag
      0.0.0.0         5.5.5.5         107         0x80000001 0x00A4F9 1

## ** configuration **

---
      interface Loopback0
      ip address 1.1.1.1 255.255.255.255
      interface FastEthernet0/1
      no switchport
      ip address 10.10.1.1 255.255.255.0
      router ospf 1
      router-id 1.1.1.1
      log-adjacency-changes
      network 10.10.1.0 0.0.0.255 area 1

---
      interface Loopback0
      ip address 2.2.2.2 255.255.255.255
      interface FastEthernet0/2
      no switchport
      ip address 10.10.1.2 255.255.255.0
      router ospf 1
      router-id 2.2.2.2
      log-adjacency-changes
      network 10.10.1.0 0.0.0.255 area 1

---

      interface Loopback0
       ip address 3.3.3.3 255.255.255.255
      !
      interface FastEthernet0/3
       no switchport
       ip address 10.10.1.3 255.255.255.0
      !
       interface FastEthernet0/0
       no switchport
       ip address 10.10.0.3 255.255.255.0
      !
       router ospf 1
       router-id 3.3.3.3
       log-adjacency-changes
       network 10.10.1.0 0.0.0.255 area 1
       network 10.10.0.0 0.0.0.255 area 0
      !

---

      interface Loopback0
       ip address 4.4.4.4 255.255.255.255
      !
      interface FastEthernet0/0
       no switchport
       ip address 10.10.0.4 255.255.255.0
      !
       interface FastEthernet0/1
       no switchport
       ip address 10.10.2.4 255.255.255.0
      !
       router ospf 1
       router-id 4.4.4.4
       log-adjacency-changes
       network 10.10.0.0 0.0.0.255 area 0
       network 10.10.2.0 0.0.0.255 area 2
      !

---

      interface Loopback0
       ip address 5.5.5.5 255.255.255.255
      !
      !
       interface FastEthernet0/1
       no switchport
       ip address 10.10.2.5 255.255.255.0
      !
       interface FastEthernet0/0
       no switchport
       ip address 10.10.3.5 255.255.255.0
      !
      ip route 0.0.0.0 0.0.0.0 10.10.3.1
      !
       router ospf 1
       router-id 5.5.5.5
       log-adjacency-changes
       network 10.10.2.0 0.0.0.255 area 2
       default-information originate always
---


---


---


---


---


---
