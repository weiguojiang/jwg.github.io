---
layout: post
title:  bird case
category: blog
description:  bird test configuration
---

## ** introduction **
In order to learn the Bird's routing functionality,
i design one case to test it.

## ** case 1  **

### ** env description **

<img src="/images/other/bird-case.png" alt="sdn" width="1000"></a>

here i create AS 100, AS 200, AS 300, AS 500 and AS 600.  
in AS 200, there are 4 routes, each runs ospf.  
Bird-1 and R3 is ibgp neighbor. same as Bird-1 and R3.
bird-1 and R4 is the reflector client of R3.   
all the router has create one IPaddr on loopback interface and
will be the root id.   

#### ** configuration **
R1

      interface Loopback0
       ip address 1.1.1.1 255.255.255.255
      interface FastEthernet0/0
       no switchport
       ip address 10.0.11.1 255.255.255.0
      router bgp 100
       no synchronization
       no auto-summary
       neighbor 10.0.11.11 remote-as 200
       network 1.1.1.1 mask 255.255.255.255

R2

      interface Loopback0
       ip address 2.2.2.2 255.255.255.255
      interface FastEthernet0/0
       no switchport
       ip address 10.0.52.2 255.255.255.0
      router bgp 100
       no synchronization
       no auto-summary
       neighbor 10.0.52.12 remote-as 200
       network 2.2.2.2 mask 255.255.255.255

R3

      interface Loopback0
      ip address 3.3.3.3 255.255.255.255
      interface FastEthernet0/2
      no switchport
      ip address 10.0.12.3 255.255.255.0
      interface FastEthernet0/3
      no switchport
      ip address 10.0.22.3 255.255.255.0
      !
      router ospf 1
      log-adjacency-changes
      network 10.0.12.0 0.0.0.255 area 0
      network 10.0.22.0 0.0.0.255 area 0
      network 3.3.3.3 0.0.0.0 area 0
      !
      router bgp 200
      no synchronization
      no auto-summary
      neighbor 11.11.11.11 remote-as 200
      neighbor 11.11.11.11 update-source loopback 0
      neighbor 11.11.11.11 route-reflector-client
      neighbor 22.22.22.22 remote-as 200
      neighbor 22.22.22.22 update-source loopback 0
      neighbor 4.4.4.4 remote-as 200
      neighbor 4.4.4.4 update-source loopback 0
      neighbor 4.4.4.4 route-reflector-client
      network 3.3.3.3 mask 255.255.255.255

R4

      interface Loopback0
       ip address 4.4.4.4 255.255.255.255
      interface FastEthernet0/0
       no switchport
       ip address 10.0.45.4 255.255.255.0
      interface FastEthernet0/1
       no switchport
       ip address 10.0.34.4 255.255.255.0
      !
      router ospf 1
      log-adjacency-changes
      network 10.0.34.0 0.0.0.255 area 0
      network 4.4.4.4 0.0.0.0 area 0
      !
      router bgp 200
       no synchronization
       no auto-summary
       neighbor 3.3.3.3 remote-as 200
       neighbor 3.3.3.3 update-source loopback 0
       neighbor 3.3.3.3 next-hop-self
       neighbor 10.0.45.5 remote-as 600
       network 4.4.4.4 mask 255.255.255.255
       network 10.0.45.0 mask 255.255.255.0
       network 10.0.34.0 mask 255.255.255.0

Bird-1

      for num in 0 1 2 3; do sudo ip link set dev eth"$num" up;  done
      sudo ip addr add 11.11.11.11/32 dev lo
      sudo ip addr add 10.0.11.11/24 dev eth0
      sudo ip addr add 10.0.12.11/24 dev eth2

-----------------bird.conf --------------------------------------

      log syslog all;

      # Route ID
      router id 11.11.11.11;

      # "Pull UP" Route for BGP
      protocol static static_bgp {
              import all;
            #  route 10.0.12.0/24 reject;
      }

      protocol ospf MyOSPF {
             tick 2;
             rfc1583compat yes;
             area 0.0.0.0 {
                     stub no;
                     interface "eth2" {
                             priority 1;
                             type broadcast;
                             authentication none;
                     };
                     interface "lo" {
                             authentication none;
                     };
             };
      }

      # BGP Configuration
      protocol bgp {
              import all;
              export where proto = "MyOSPF";
              local as 200;
              neighbor 10.0.11.1 as 100;

      }

      protocol bgp {
              import all;
              export where proto = "static_bgp";

              local as 200;
              neighbor 3.3.3.3 as 200;
              next hop self;
              source address 11.11.11.11;
      }
---------------end for .conf

    vim /usr/local/etc/bird.conf
    sudo kill -9 `ps -ef | grep bird | grep root | awk '{print $1}'`
    sudo bird


### ** status in router **
Lets check the R1 state.  

      R1#show ip bgp
      BGP table version is 13, local router ID is 1.1.1.1
      Status codes: s suppressed, d damped, h history, * valid, > best, i - internal,
                    r RIB-failure, S Stale
      Origin codes: i - IGP, e - EGP, ? - incomplete

         Network          Next Hop            Metric LocPrf Weight Path
      *> 1.1.1.1/32       0.0.0.0                  0         32768 i
      *> 3.3.3.3/32       10.0.11.11                             0 200 i
      *> 10.0.22.0/24     10.0.11.11                             0 200 i
      *> 11.11.11.11/32   10.0.11.11                             0 200 i
      R1#
      R1#show ip route bgp
           3.0.0.0/32 is subnetted, 1 subnets
      B       3.3.3.3 [20/0] via 10.0.11.11, 00:45:48
           10.0.0.0/24 is subnetted, 2 subnets
      B       10.0.22.0 [20/0] via 10.0.11.11, 00:45:48
           11.0.0.0/32 is subnetted, 1 subnets
      B       11.11.11.11 [20/0] via 10.0.11.11, 00:46:44
      R1#
      R1#
      R1#ping 3.3.3.3

      Type escape sequence to abort.
      Sending 5, 100-byte ICMP Echos to 3.3.3.3, timeout is 2 seconds:
      .....
      Success rate is 0 percent (0/5)

Lets check the BIRD-1 state, we will use birdc client

      gns3@box:~$ sudo birdc
      BIRD 1.5.0 ready.
      bird>

if we want to check our neighbor BiRD gives as a bunch of information

      bird> show protocols all bgp2
      name     proto    table    state  since       info
      bgp2     BGP      master   up     15:39:47    Established
        Preference:     100
        Input filter:   ACCEPT
        Output filter:  REJECT
        Routes:         1 imported, 0 exported, 0 preferred
        Route change stats:     received   rejected   filtered    ignored   accepted
          Import updates:              1          0          0          0          1
          Import withdraws:            0          0        ---          0          0
          Export updates:              5          0          5        ---          0
          Export withdraws:            0        ---        ---        ---          0
        BGP state:          Established
          Neighbor address: 3.3.3.3
          Neighbor AS:      200
          Neighbor ID:      3.3.3.3
          Neighbor caps:    refresh
          Session:          internal multihop
          Source address:   11.11.11.11
          Hold timer:       128/180
          Keepalive timer:  56/60

Show routes in RIB from BGP

      bird> show route protocol bgp1
      1.1.1.1/32         via 10.0.11.1 on eth0 [bgp1 15:38:45] * (100) [AS100i]
      bird> show route protocol bgp2
      3.3.3.3/32         via 10.0.12.3 on eth2 [bgp2 15:40:36 from 3.3.3.3] (100/11) [i]
      bird> show route protocol MyOSPF
      3.3.3.3/32         via 10.0.12.3 on eth2 [MyOSPF 15:39:41] * I (150/11) [3.3.3.3]
      11.11.11.11/32     dev lo [MyOSPF 15:38:41] * I (150/0) [11.11.11.11]
      10.0.12.0/24       dev eth2 [MyOSPF 15:39:29] I (150/10) [11.11.11.11]
      10.0.22.0/24       via 10.0.12.3 on eth2 [MyOSPF 15:39:41] * I (150/11) [3.3.3.3]

Show route exported to kernel, then based on kernel’s weight of the protocol they are inserted into FIB

      bird> show route export kernel1
      1.1.1.1/32         via 10.0.11.1 on eth0 [bgp1 15:38:45] * (100) [AS100i]
      3.3.3.3/32         via 10.0.12.3 on eth2 [MyOSPF 15:39:41] * I (150/11) [3.3.3.3]
      10.0.12.0/24       unreachable [static_bgp 15:38:40] ! (200)
      10.0.22.0/24       via 10.0.12.3 on eth2 [MyOSPF 15:39:41] * I (150/11) [3.3.3.3]


### ** comment for bird configuration **  
general, it's not easy to do it in bird.conf

 * not know how to do "no synchronization" in the bird.conf
 * not know how to do "network 5.5.5.5 mask 255.255.255.255"

not know the reason for the following result:

 * Notice that Bird-1 doesn't advertise any route info to R3.
 * Notice that Bird-1 doesn't advertise R5  route info to R1. 

### ** left work **
 * in R1, it fails to ping R3. I will check it next day

## ** reference list **
 * [bird-protocol-bgp](http://bird.network.cz/?get_doc&f=bird-6.html#ss6.2)
