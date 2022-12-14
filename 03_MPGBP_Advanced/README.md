# MPBGP Advanced Guide Commands

# RouterA

```
configure terminal

interface f0/0
ip address 192.3.1.10 255.255.255.0
no shutdown

interface f0/1
ip address 192.3.2.10 255.255.255.0
no shutdown

interface f1/0
ip address 200.10.1.10 255.255.255.0
no shutdown

interface f1/1
ip address 200.10.11.10 255.255.255.0
no shutdown

interface f2/0
ip address 200.0.0.10 255.255.255.0
no shutdown
```

## BGP IPv4

```
router bgp 300
address-family ipv4 unicast
neighbor 200.10.1.1 remote-as 200
neighbor 200.10.11.11 remote-as 100
neighbor 200.0.0.11 remote-as 100
network 192.3.1.0 mask 255.255.255.0
network 192.3.2.0 mask 255.255.255.0
```

## Communities

Communities can be used to label route announcements using multiple criteria, the label allows to filter or assign attribute values accordingly.

```
ip bgp-community new-format
route-map changeClink1 permit 10
set community 300:1
route-map changeClink2 permit 10
set community 300:2

router bgp 300
address-family ipv4 unicast
neighbor 200.10.11.11 route-map changeClink1 out
neighbor 200.10.11.11 send-community
neighbor 200.0.0.11 route-map changeClink2 out
neighbor 200.0.0.11 send-community
```

# RouterB

```
configure terminal

interface f0/0
ip address 192.1.1.11 255.255.255.128
no shutdown

interface f0/1
ip address 192.1.1.139 255.255.255.128
no shutdown

interface f1/0
ip address 200.2.11.11 255.255.255.0
no shutdown

interface f1/1
ip address 200.10.11.11 255.255.255.0
no shutdown

interface f2/0
ip address 200.0.0.11 255.255.255.0
no shutdown
```

## BGP IPv4

```
router bgp 100
address-family ipv4 unicast
neighbor 200.2.11.2 remote-as 200
neighbor 200.10.11.10 remote-as 300
neighbor 200.0.0.10 remote-as 300
network 192.1.1.0 mask 255.255.255.128
network 192.1.1.128 mask 255.255.255.128
```

## Communities

```
config terminal
ip bgp-community new-format
router bgp 100
address-family ipv4 unicast
neighbor 200.2.11.2 send-community
```

# Router1

```
configure terminal

interface f0/0
ip address 192.2.12.1 255.255.255.0
no shutdown

interface f0/1
ip address 192.2.13.1 255.255.255.0
no shutdown

interface f1/0
ip address 200.10.1.1 255.255.255.0
no shutdown

interface Lo0
ip address 192.2.0.1 255.255.255.255
no shutdown
```

## BGP IPv4

```
router bgp 200
address-family ipv4 unicast
neighbor 192.2.12.2 remote-as 200
neighbor 192.2.13.3 remote-as 200
neighbor 200.10.1.10 remote-as 300
neighbor 192.2.12.2 next-hop-self
neighbor 192.2.13.3 next-hop-self
!network 192.2.12.0 mask 255.255.255.0
!network 192.2.13.0 mask 255.255.255.0
!network 192.2.0.1 mask 255.255.255.255
```

## eBGP and iBGP with OSPF

```
router ospf 100
network 192.2.12.0 0.0.0.255 area 0
network 192.2.13.0 0.0.0.255 area 0
network 192.2.0.1 0.0.0.0 area 0
redistribute bgp 200 subnets
```

## Redistribution of OSPF routes into BGP

```
router bgp 200
address-family ipv4 unicast
redistribute ospf 100
```

## Establish Neighbor relations between Loopback interfaces

```
router bgp 200
address-family ipv4 unicast
no neighbor 192.2.12.2 remote-as 200
neighbor 192.2.0.2 remote-as 200
neighbor 192.2.0.2 next-hop-self
exit
neighbor 192.2.0.2 update-source Loopback 0
```

## EBGP, IBGP and OSPF (relative) administrative distances

Note: by default EBGP and IBGP route announcements have an administrative distance of 20 and 200, respectively. OSPF routes have an administrative distance of 110.

```
router ospf 100
distance 220
```

## Route Maps

```
ip as-path access-list 1 permit ^$
route-map routes-out
match as-path 1
router bgp 200
address-family ipv4 unicast
neighbor 200.10.1.10 route-map routes-out out
```

## BGP conflicts with IGP routing

```
router ospf 100
no redistribute bgp 200 subnets
no redistribute bgp 200
default-information originate always metric 5
```

## Router 1 as BGP's preferred exit from the AS

```
router bgp 200
bgp default local-preference 200
```

## BGP Neighbor relations over IP-IP tunnels

```
interface Tunnel 0
ip address 10.0.0.1 255.255.255.252
tunnel source Loopback 0
tunnel destination 192.2.0.2
tunnel mode ipip

router bgp 200
address-family ipv4 unicast
no neighbor 192.2.0.2 remote-as 200
neighbor 10.0.0.2 remote-as 200
neighbor 10.0.0.2 next-hop-self
```

# Router2

```
configure terminal

interface f0/0
ip address 192.2.12.2 255.255.255.0
no shutdown

interface f0/1
ip address 192.2.23.2 255.255.255.0
no shutdown

interface f1/0
ip address 200.2.11.2 255.255.255.0
no shutdown

interface Lo0
ip address 192.2.0.2 255.255.255.255
no shutdown
```

## BGP IPv4

```
router bgp 200
address-family ipv4 unicast
neighbor 192.2.12.1 remote-as 200
neighbor 192.2.23.3 remote-as 200
neighbor 200.2.11.11 remote-as 100
neighbor 192.2.12.1 next-hop-self
neighbor 192.2.23.3 next-hop-self
!network 192.2.12.0 mask 255.255.255.0
!network 192.2.23.0 mask 255.255.255.0
!network 192.2.0.2 mask 255.255.255.255
```

## eBGP and iBGP with OSPF

```
router ospf 100
network 192.2.12.0 0.0.0.255 area 0
network 192.2.23.0 0.0.0.255 area 0
network 192.2.0.2 0.0.0.0 area 0
redistribute bgp 200 subnets
```

## Redistribution of OSPF routes into BGP

```
router bgp 200
address-family ipv4 unicast
redistribute ospf 100
```

## Establish Neighbor relations between Loopback interfaces

```
router bgp 200
address-family ipv4 unicast
no neighbor 192.2.12.1 remote-as 200
neighbor 192.2.0.1 remote-as 200
neighbor 192.2.0.1 next-hop-self
exit
neighbor 192.2.0.1 update-source Loopback 0
```

## Route Maps

```
ip as-path access-list 1 permit ^$
route-map routes-out
match as-path 1
router bgp 200
address-family ipv4 unicast
neighbor 200.2.11.11 route-map routes-out out
```

## Local Preference based on Community value

Local preference attribute can be used to chose (within the AS) between multiple routes to the same destination.

```
ip bgp-community new-format
ip community-list 1 permit 300:1
ip community-list 2 permit 300:2
route-map routes-in permit 10
match community 1
set local-preference 22
route-map routes-in permit 20
match community 2
set local-preference 111
router bgp 200
address-family ipv4 unicast
neighbor 200.2.11.11 route-map routes-in in
```

## Remove the local-preference changing route-map

```
router bgp 200
no neighbor 200.2.11.11 route-map routes-in in
```

## BGP conflicts with IGP routing

```
router ospf 100
no redistribute bgp 200 subnets
default-information originate always metric 10
```

## BGP Neighbor relations over IP-IP tunnels

```
interface Tunnel 0
ip address 10.0.0.2 255.255.255.252
tunnel source Loopback 0
tunnel destination 192.2.0.1
tunnel mode ipip

router bgp 200
address-family ipv4 unicast
no neighbor 192.2.0.1 remote-as 200
neighbor 10.0.0.1 remote-as 200
neighbor 10.0.0.1 next-hop-self
```

# Router3

```
configure terminal

interface f0/0
ip address 192.2.13.3 255.255.255.0
no shutdown

interface f0/1
ip address 192.2.23.3 255.255.255.0
no shutdown
```

## BGP IPv4

```
router bgp 200
address-family ipv4 unicast
neighbor 192.2.13.1 remote-as 200
neighbor 192.2.23.2 remote-as 200
network 192.2.13.0 mask 255.255.255.0
network 192.2.23.0 mask 255.255.255.0
```

## eBGP and iBGP with OSPF

```
router ospf 100
network 192.2.13.0 0.0.0.255 area 0
network 192.2.23.0 0.0.0.255 area 0
```