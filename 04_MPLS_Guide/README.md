# MPLS Guide

## Router A

```
configure terminal

interface f0/0
ip address 192.3.1.10 255.255.255.0
ip ospf 1 area 0
no shutdown

interface f0/1
ip address 192.3.2.10 255.255.255.0
ip ospf 1 area 0
no shutdown

interface f1/0
ip address 200.10.1.10 255.255.255.0
ip ospf 1 area 0
no shutdown

interface f1/1
ip address 200.10.2.10 255.255.255.0
ip ospf 1 area 0
no shutdown

interface Lo0
ip address 192.2.0.10 255.255.255.255
ip ospf 1 area 0
no shutdown
```

### Tunnel between RouterA and RouterB

> RSVP is used to establish and maintain LSP tunnels based on the calculated path using **PATH** and **RSVP RESV** messages. The RSVP protocol specification has been extended so that the RESV messages also distribute label information.

```
configure terminal

interface tunnel 1
ip unnumbered Loopback0
tunnel destination 192.2.0.11
tunnel mode mpls traffic-eng
tunnel mpls traffic-eng bandwidth 150
tunnel mpls traffic-eng path-option 1 explicit name path1

interface tunnel 2
ip unnumbered Loopback0
tunnel destination 192.2.0.11
tunnel mode mpls traffic-eng
tunnel mpls traffic-eng bandwidth 150
tunnel mpls traffic-eng path-option 1 explicit name path2

ip explicit-path name path1 enable
next-address 200.10.1.1
next-address 200.1.11.11

ip explicit-path name path2 enable
next-address 200.10.2.2
next-address 200.2.11.11
```

### Static routes: netB.1 via tunnel 1, netB.2 via tunnel 2

```
configure terminal

ip route 192.1.1.0 255.255.255.128 tunnel1
ip route 192.1.1.128 255.255.255.128 tunnel2
```

### Autoroute announce

> The `tunnel mpls traffic-eng autoroute announce` command announces the presence of a tunnel via routing protocol.

```
configure terminal

interface tunnel 1
tunnel mpls traffic-eng autoroute announce
tunnel mpls traffic-eng autoroute metric 20

interface tunnel 2
tunnel mpls traffic-eng autoroute announce
tunnel mpls traffic-eng autoroute metric 20
```

### Dynamic tunnels between RouterA and RouterB

```
configure terminal

interface tunnel 3
ip unnumbered Loopback0
tunnel destination 192.2.0.11
tunnel mode mpls traffic-eng
tunnel mpls traffic-eng autoroute announce
tunnel mpls traffic-eng bandwidth 150
tunnel mpls traffic-eng path-option 1 dynamic

interface tunnel 4
ip unnumbered Loopback0
tunnel destination 192.2.0.11
tunnel mode mpls traffic-eng
tunnel mpls traffic-eng autoroute announce
tunnel mpls traffic-eng auto-bw
tunnel mpls traffic-eng path-option 1 dynamic
```

### Disable tunnels in RouterA

```
configure terminal

interface range tunnel 1-4
shutdown
```

### MPLS-VPN (with MP-BGP and VRF)

```
configure terminal

mpls ip

ip vrf VPN-1
rd 200:1
route-target export 200:1
route-target import 200:1

ip vrf VPN-2
rd 200:2
route-target export 200:2
route-target import 200:2

interface f0/0
ip vrf forwarding VPN-1
ip address 192.3.1.10 255.255.255.0

interface f0/1
ip vrf forwarding VPN-2
ip address 192.3.2.10 255.255.255.0

interface f1/0
mpls ip
interface f1/1
mpls ip

router bgp 200
bgp router-id 10.10.10.10
neighbor 192.2.0.11 remote-as 200
neighbor 192.2.0.11 update-source Loopback0

address-family vpnv4
neighbor 192.2.0.11 activate
neighbor 192.2.0.11 send-community both

address-family ipv4 vrf VPN-1
redistribute connected

address-family ipv4 vrf VPN-2
redistribute connected
```

### Default route from the VPN-1 to Router1

```
configure terminal

ip route vrf VPN-1 0.0.0.0 0.0.0.0 200.10.1.1 global
ip route 192.3.1.0 255.255.255.0 FastEthernet0/0

router ospf 1
redistribute static subnets
```

## Router B

```
configure terminal

interface f0/0
ip address 192.1.1.11 255.255.255.128
ip ospf 1 area 0
no shutdown

interface f0/1
ip address 192.1.1.139 255.255.255.128
ip ospf 1 area 0
no shutdown

interface f1/0
ip address 200.1.11.11 255.255.255.0
ip ospf 1 area 0
no shutdown

interface f1/1
ip address 200.2.11.11 255.255.255.0
ip ospf 1 area 0
no shutdown

interface Lo0
ip address 192.2.0.11 255.255.255.255
ip ospf 1 area 0
no shutdown
```

### Tunnel between RouterB and RouterA

> RSVP is used to establish and maintain LSP tunnels based on the calculated path using **PATH** and **RSVP RESV** messages. The RSVP protocol specification has been extended so that the RESV messages also distribute label information.

```
configure terminal

interface tunnel 1
ip unnumbered Loopback0
tunnel destination 192.2.0.10
tunnel mode mpls traffic-eng
tunnel mpls traffic-eng bandwidth 150
tunnel mpls traffic-eng path-option 1 explicit name path1

interface tunnel 2
ip unnumbered Loopback0
tunnel destination 192.2.0.10
tunnel mode mpls traffic-eng
tunnel mpls traffic-eng bandwidth 150
tunnel mpls traffic-eng path-option 1 explicit name path2

ip explicit-path name path1 enable
next-address 200.1.11.1
next-address 200.10.1.1

ip explicit-path name path2 enable
next-address 200.2.11.2
next-address 200.10.2.1
```

### Static routes: netA.1 via tunnel 1, netA.2 via tunnel 2

```
configure terminal

ip route 192.3.1.0 255.255.255.0 tunnel1
ip route 192.3.2.0 255.255.255.0 tunnel2
```

### Autoroute announce

> The `tunnel mpls traffic-eng autoroute announce` command announces the presence of a tunnel via routing protocol.

```
configure terminal

interface tunnel 1
tunnel mpls traffic-eng autoroute announce

interface tunnel 2
tunnel mpls traffic-eng autoroute announce
```

### Dynamic tunnels between RouterA and RouterB

```
configure terminal

interface tunnel 3
ip unnumbered Loopback0
tunnel destination 192.2.0.10
tunnel mode mpls traffic-eng
tunnel mpls traffic-eng autoroute announce
tunnel mpls traffic-eng bandwidth 150
tunnel mpls traffic-eng path-option 1 dynamic

interface tunnel 4
ip unnumbered Loopback0
tunnel destination 192.2.0.10
tunnel mode mpls traffic-eng
tunnel mpls traffic-eng autoroute announce
tunnel mpls traffic-eng auto-bw
tunnel mpls traffic-eng path-option 1 dynamic
```

### MPLS-VPN (with MP-BGP and VRF)

```
configure terminal

mpls ip

ip vrf VPN-1
rd 200:1
route-target export 200:1
route-target import 200:1

ip vrf VPN-2
rd 200:2
route-target export 200:2
route-target import 200:2

interface f0/0
ip vrf forwarding VPN-1
ip address 192.1.1.11 255.255.255.128

interface f0/1
ip vrf forwarding VPN-2
ip address 192.1.1.139 255.255.255.128

interface f1/0
mpls ip
interface f1/1
mpls ip

router bgp 200
bgp router-id 11.11.11.11
neighbor 192.2.0.10 remote-as 200
neighbor 192.2.0.10 update-source Loopback0

address-family vpnv4
neighbor 192.2.0.10 activate
neighbor 192.2.0.10 send-community both

address-family ipv4 vrf VPN-1
redistribute connected

address-family ipv4 vrf VPN-2
redistribute connected
```


## Router 1

```
configure terminal

interface f0/0
ip address 200.10.1.1 255.255.255.0
ip ospf 1 area 0
no shutdown

interface f0/1
ip address 200.1.2.1 255.255.255.0
ip ospf 1 area 0
no shutdown

interface f1/0
ip address 200.1.11.1 255.255.255.0
ip ospf 1 area 0
no shutdown

interface Lo0
ip address 200.2.0.1 255.255.255.255
ip ospf 1 area 0
no shutdown
```

## Router 2

```
configure terminal

interface f0/0
ip address 200.10.2.2 255.255.255.0
ip ospf 1 area 0
no shutdown

interface f0/1
ip address 200.1.2.2 255.255.255.0
ip ospf 1 area 0
no shutdown

interface f1/0
ip address 200.2.11.2 255.255.255.0
ip ospf 1 area 0
no shutdown

interface Lo0
ip address 200.2.0.2 255.255.255.255
ip ospf 1 area 0
no shutdown
```

## All routers

### Enable CEP and MPLS (LDP)

> **Note 1:** The LSRs must have (up) Loopback interfaces with an address mask of 32 bits and these interfaces must be reachable with the global IP routing table.
> **Note 2:** When you activate MPLS, LDP is automatically turned on and labels start being advertised (default mode: downstream unsolicited).
> **Note 3:** Cisco's routers have by default MPLS Penultimate Hop Popping (PHP) mechanism active.

```
configure terminal
ip cef
mpls ip

interface f0/0
mpls ip
interface f0/1
mpls ip
interface f1/0
mpls ip
interface f1/1
mpls ip
```

### Disable MPLS (LDP)

```
configure terminal
no mpls ip

interface f0/0
no mpls ip
interface f0/1
no mpls ip
interface f1/0
no mpls ip
interface f1/1
no mpls ip
```

### Enable MPLS (RSVP-TE)

```
configure terminal
mpls traffic-eng tunnels

interface f0/0
mpls traffic-eng tunnels
interface f0/1
mpls traffic-eng tunnels
interface f1/0
mpls traffic-eng tunnels
interface f1/1
mpls traffic-eng tunnels
```

### Enable traffic engineering features on OSPF

> The `show ip ospf mpls traffic-eng link` command shows the links advertised by OSPF at a given router, including the RSVP characteristics. 
> The `show ip ospf database opaque-area` command shows the OSPF database restricted part corresponding to Type 10 LSAs, showing the database that is used by the MPLS TE process to calculate TE routes for tunnels.

```
configure terminal
router ospf 1
mpls traffic-eng area 0
mpls traffic-eng router-id Loopback 0
```

### Enable RSVP

```
configure terminal

interface f0/0
ip rsvp bandwidth 512 512
interface f0/1
ip rsvp bandwidth 512 512
interface f1/0
ip rsvp bandwidth 512 512
interface f1/1
ip rsvp bandwidth 512 512
```

### Disable relevant MPLS (RSVP-TE) and OSPF commands in all Routers

```
configure terminal
no mpls traffic-eng tunnels

interface range FastEthernet 0/0-1
no mpls traffic-eng tunnels
no ip rsvp bandwidth 512 512

interface range FastEthernet 1/0-1
no mpls traffic-eng tunnels
no ip rsvp bandwidth 512 512

router ospf 1
no mpls traffic-eng router-id Loopback0
no mpls traffic-eng area 0
```