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
```

## BGP IPv4

```
router bgp 300
address-family ipv4 unicast
neighbor 200.10.1.1 remote-as 200
neighbor 200.10.11.11 remote-as 100
network 192.3.1.0 mask 255.255.255.0
network 192.3.2.0 mask 255.255.255.0
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
```

## BGP IPv4

```
router bgp 100
address-family ipv4 unicast
neighbor 200.2.11.2 remote-as 200
neighbor 200.10.11.10 remote-as 300
network 192.1.1.0 mask 255.255.255.128
network 192.1.1.128 mask 255.255.255.128
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