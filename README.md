# eBGP

BGP is a routing protocol used to exchange information between autonomous systems. An autonomous system is defined as a collection of networks under a single technical administration domain.

As an exterior gateway protocol, BGP provides interdomain routing between autonomous systems.

## BGP data structures 

BGP neighbor table: For BGP to establish an adjacency, it must be explicitly configured with each neighbor. BGP forms a TCP relationship with each of the configured neighbors and keeps track of the state of these relationships by periodically sending a BGP/TCP keepalive message.

BGP table: After establishing an adjacency, the neighbors exchange the BGP routes. Each router collects these routes from each neighbor that successfully establishes an adjacency and then places the routes in its BGP forwarding database. All routes that have been learned from each neighbor are placed into the BGP forwarding database. The best route for each network is selected from the BGP forwarding database using the BGP route selection process and is then offered to the IP routing table.

IP routing table: Each router compares the offered BGP routes to any other possible paths to those networks, and the best route—based on administrative distance—is installed in the IP routing table. External BGP routes (BGP routes that are learned from an external AS) have an administrative distance of 20. Internal BGP routes (BGP routes that are learned from within the AS) have an administrative distance of 200. 

## Path Selection

After BGP receives updates about different destinations from different autonomous systems, it chooses the single best path to reach a specific destination.

Routing policy is based on the following attributes:

Prefer the highest weight attribute (local to router)
Prefer the highest local preference attribute (global with AS)
Prefer route originated by the local router (next hop = 0.0.0.0)
Prefer the shortest AS path (least number of autonomous systems in AS-path attribute)
Prefer the lowest origin attribute (IGP < EGP < incomplete)
Prefer the lowest MED attribute (exchanged between autonomous systems)
Prefer an EBGP path over an IBGP path
(IBGP route) Prefer the path through the closest IGP neighbor (best IGP metric)
(EBGP route) Prefer the oldest EBGP path (neighbor with the longest uptime)
Prefer the path with the lowest neighbor BGP router ID
Prefer the path with the lowest neighbor IP address (multiple paths to the same neighbor)

When faced with multiple routes to the same destination, BGP chooses the best route for routing traffic toward the destination by following the route selection process. 

## eBGP Configuration

![Creating a VLAN](https://raw.githubusercontent.com/deliawolf/eBGP/main/OSPF%20-%20eBGP%20-%20EIGRP.drawio.png)

here i am going to provide config for whole topology
- OSPF
- EIGRP
- and last is eBGP
the reasion why is because it is to prevent missunderstanding or missconfiguration.

### OSPF Routers Configuration

1. Router 1 (1.1.1.1)

Interfaces Configuration:
```
!
interface GigabitEthernet0/0
 ip address 10.10.1.1 255.255.255.0
 duplex auto
 speed auto
 media-type rj45
!
interface GigabitEthernet0/1
 ip address 10.10.101.1 255.255.255.0
 duplex auto
 speed auto
 media-type rj45
!
interface GigabitEthernet0/2
 ip address 10.10.100.1 255.255.255.0
 duplex auto
 speed auto
 media-type rj45
!
```
OSPF Configuration:
```
!
router ospf 100
 router-id 1.1.1.1
 network 10.10.100.0 0.0.0.255 area 1
 network 10.10.101.0 0.0.0.255 area 2
 default-information originate always
!
```
eBGP Configruation:
```
!
router bgp 100
 bgp router-id 1.1.1.1
 bgp log-neighbor-changes
 redistribute ospf 100
 neighbor 10.10.1.2 remote-as 1
!
```
2. Router 2 (1.1.1.2)

Interfaces Configuration
```
!
interface GigabitEthernet0/0
 ip address 10.10.100.2 255.255.255.0
 duplex auto
 speed auto
 media-type rj45
!
interface GigabitEthernet0/1
 ip address 10.10.102.2 255.255.255.0
 duplex auto
 speed auto
 media-type rj45
!
```
OSPF Configuration
```
!
router ospf 200
 router-id 1.1.1.2
 network 10.10.100.0 0.0.0.255 area 1
 network 10.10.102.0 0.0.0.255 area 0
!
```
3. Router 3 (1.1.1.3)

Interfaces Configuration:
```
!
interface GigabitEthernet0/0
 ip address 10.10.101.2 255.255.255.0
 duplex auto
 speed auto
 media-type rj45
!
interface GigabitEthernet0/1
 ip address 10.10.102.1 255.255.255.0
 duplex auto
 speed auto
 media-type rj45
!
```
OSPF COnfiguration:
```
!
router ospf 300
 router-id 1.1.1.3
 network 10.10.101.0 0.0.0.255 area 2
 network 10.10.102.0 0.0.0.255 area 0
!
```

###  EIGRP Routers Configuration

1. Router 1 (3.3.3.3)

Interfaces Configuration:
```
!
interface GigabitEthernet0/0
 ip address 10.10.2.1 255.255.255.0
 duplex auto
 speed auto
 media-type rj45
!
interface GigabitEthernet0/1
 ip address 10.10.201.1 255.255.255.0
 ip summary-address eigrp 200 0.0.0.0 0.0.0.0
 duplex auto
 speed auto
 media-type rj45
!
interface GigabitEthernet0/2
 ip address 10.10.200.1 255.255.255.0
 ip summary-address eigrp 200 0.0.0.0 0.0.0.0
 duplex auto
 speed auto
 media-type rj45
!
```
EIGRP Configuration:
```
!
router eigrp 200
 network 10.10.200.0 0.0.0.255
 network 10.10.201.0 0.0.0.255
 eigrp router-id 3.3.3.3
!
```
eBGP COnfiguration:
```
!
router bgp 200
 bgp router-id 3.3.3.3
 bgp log-neighbor-changes
 redistribute eigrp 200
 neighbor 10.10.2.2 remote-as 1
!
```

2. Router 2(3.3.3.4)

Interfaces Configuration:
```
!
interface GigabitEthernet0/0
 ip address 10.10.201.2 255.255.255.0
 duplex auto
 speed auto
 media-type rj45
!
interface GigabitEthernet0/1
 ip address 10.10.202.1 255.255.255.0
 duplex auto
 speed auto
 media-type rj45
!
```
EIGRP Configuration:
```
!
router eigrp 200
 network 10.10.201.0 0.0.0.255
 network 10.10.202.0 0.0.0.255
 eigrp router-id 3.3.3.4
!
```

3. Router 3 (3.3.3.5)

Interfaces Configuration:
```
!
interface GigabitEthernet0/0
 ip address 10.10.200.2 255.255.255.0
 duplex auto
 speed auto
 media-type rj45
!
interface GigabitEthernet0/1
 ip address 10.10.202.2 255.255.255.0
 duplex auto
 speed auto
 media-type rj45
!
```
EIGRP Configuration:
```
!
router eigrp 200
 network 10.10.200.0 0.0.0.255
 network 10.10.202.0 0.0.0.255
 eigrp router-id 3.3.3.5
!
```
### eBGP Configuration

Router 1 (2.2.2.2)

Interfaces Configuration:
```
!
interface GigabitEthernet0/0
 ip address 10.10.1.2 255.255.255.0
 duplex auto
 speed auto
 media-type rj45
!
interface GigabitEthernet0/1
 ip address 10.10.2.2 255.255.255.0
 duplex auto
 speed auto
 media-type rj45
!
```
eBGP Configuration:
in eBGP configuration there mandatory config we need to have:
1. the network
2. neighbor
3. redistribution

without 3 of those things, the bgp will not work and ping communication inter-AS will not prapagated.
```
!
router bgp 1
 bgp router-id 2.2.2.2
 bgp log-neighbor-changes
 network 10.10.1.0 mask 255.255.255.0
 network 10.10.2.0 mask 255.255.255.0
 redistribute ospf 100 match internal external 1 external 2
 redistribute eigrp 200
 neighbor 10.10.1.1 remote-as 100
 neighbor 10.10.2.1 remote-as 200
!
```
there also importand config in ospf and eigrp that is dfault gateway.
for example in Router 1 OSPF have below config
```
default-information originate always
```
and in eigrp have this in eigrp interfaces
```
ip summary-address eigrp 200 0.0.0.0 0.0.0.0
```
those config make sure internal network can reach outside network, it also help propagate the rBGP to reach both internal network.

#### Testing

in here i am going to show 'show ip route' for 3 routers
1. Router 1 (OSPF)
2. Router 1 (eBGP)
3. ROuter 1 (EIGRP)
in the end we will test with ping, the ping itself is from
1. OSPF to eBGP (ping 10.10.1.2)
2. EIGRP to eBGP (ping 10.10.2.2)
3. OSPF to EIGRP (ping 10.10.202.1)
4. EIGRP to OSPF (ping 10.10.102.1)

##### SHOW IP ROUTE

1. Router 1 (OSPF)
```
Gateway of last resort is not set

      10.0.0.0/8 is variably subnetted, 11 subnets, 2 masks
C        10.10.1.0/24 is directly connected, GigabitEthernet0/0
L        10.10.1.1/32 is directly connected, GigabitEthernet0/0
B        10.10.2.0/24 [20/0] via 10.10.1.2, 02:06:12
C        10.10.100.0/24 is directly connected, GigabitEthernet0/2
L        10.10.100.1/32 is directly connected, GigabitEthernet0/2
C        10.10.101.0/24 is directly connected, GigabitEthernet0/1
L        10.10.101.1/32 is directly connected, GigabitEthernet0/1
O IA     10.10.102.0/24 [110/2] via 10.10.101.2, 15:40:15, GigabitEthernet0/1
                        [110/2] via 10.10.100.2, 15:41:03, GigabitEthernet0/2
B        10.10.200.0/24 [20/0] via 10.10.1.2, 16:04:23
B        10.10.201.0/24 [20/0] via 10.10.1.2, 16:04:23
B        10.10.202.0/24 [20/0] via 10.10.1.2, 02:15:17
```
2. Router 1 (eBGP)
```
Gateway of last resort is not set

      10.0.0.0/8 is variably subnetted, 10 subnets, 2 masks
C        10.10.1.0/24 is directly connected, GigabitEthernet0/0
L        10.10.1.2/32 is directly connected, GigabitEthernet0/0
C        10.10.2.0/24 is directly connected, GigabitEthernet0/1
L        10.10.2.2/32 is directly connected, GigabitEthernet0/1
B        10.10.100.0/24 [20/0] via 10.10.1.1, 16:02:17
B        10.10.101.0/24 [20/0] via 10.10.1.1, 16:02:17
B        10.10.102.0/24 [20/2] via 10.10.1.1, 16:02:17
B        10.10.200.0/24 [20/0] via 10.10.2.1, 16:02:48
B        10.10.201.0/24 [20/0] via 10.10.2.1, 16:02:48
B        10.10.202.0/24 [20/3072] via 10.10.2.1, 02:16:25
```
3. Router 1 (EIGRP)
```
Gateway of last resort is 0.0.0.0 to network 0.0.0.0

D*    0.0.0.0/0 is a summary, 01:46:16, Null0
      10.0.0.0/8 is variably subnetted, 11 subnets, 2 masks
B        10.10.1.0/24 [20/0] via 10.10.2.2, 02:08:58
C        10.10.2.0/24 is directly connected, GigabitEthernet0/0
L        10.10.2.1/32 is directly connected, GigabitEthernet0/0
B        10.10.100.0/24 [20/0] via 10.10.2.2, 16:06:40
B        10.10.101.0/24 [20/0] via 10.10.2.2, 16:06:40
B        10.10.102.0/24 [20/0] via 10.10.2.2, 16:06:40
C        10.10.200.0/24 is directly connected, GigabitEthernet0/2
L        10.10.200.1/32 is directly connected, GigabitEthernet0/2
C        10.10.201.0/24 is directly connected, GigabitEthernet0/1
L        10.10.201.1/32 is directly connected, GigabitEthernet0/1
D        10.10.202.0/24 
           [90/3072] via 10.10.201.2, 02:17:59, GigabitEthernet0/1
           [90/3072] via 10.10.200.2, 02:17:59, GigabitEthernet0/2
```
##### PING Result

1. OSPF to eBGP (ping 10.10.1.2)
```
inserthostname_here#ping 10.10.1.2
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 10.10.1.2, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 12/13/15 ms
```
2. EIGRP to eBGP (ping 10.10.2.2)
```
inserthostname_here#ping 10.10.2.2
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 10.10.2.2, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 12/13/16 ms
```
3. OSPF to EIGRP (ping 10.10.202.1)
```
inserthostname_here#ping 10.10.202.1
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 10.10.202.1, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 10/10/12 ms
```
4. EIGRP to OSPF (ping 10.10.102.1)
```
inserthostname_here#ping 10.10.102.1
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 10.10.102.1, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 10/10/12 ms
```

That's all the eBGP config as bridge between OSPF and EIGRP
**future plan is to cretae summarization for both ospf and eigrp so the routing table will not having to much routing path


#### Manipulate weight

in BGP locally we can manipulate path using weight but it is only on Cisco devices.
```
R1(config)#router bgp 1
R1(config-router)#neighbor 192.168.13.3 weight 500
```
