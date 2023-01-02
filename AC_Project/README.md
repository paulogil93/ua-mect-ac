# Final Project

## Start procedure

### Ping Client X interfaces

> In order to each PE router learn the MAC addresses of the terminals in each network, ping from each
terminal the respective PE (gateway)

```
Client X1: ping 10.2.1.1
Client X2: ping 10.2.2.1
Client X3: ping 10.2.3.1
```

## IP planning

```
> Available: 10.0.0.0/23

Loopbacks:
- DC.P1: 10.0.0.1/32
- DC.P2: 10.0.0.2/32
- DC.L1: 10.0.0.3/32
- DC.A1: 10.0.0.4/32
- DC.A2: 10.0.0.5/32
- DC.C1: 10.0.0.6/32

Interconnections:
- DC.P1-Porto: 10.0.1.0/30
- DC.P2-Porto: 10.0.1.4/30
- Porto-Aveiro: 10.0.1.8/30
- Porto-Coimbra: 10.0.1.12/30
- DC.L1-Lisboa: 10.0.1.16/30
- Lisboa-Coimbra: 10.0.1.20/30
- DC.A1-Aveiro: 10.0.1.24/30
- DC.A2-Aveiro: 10.0.1.28/30
- Aveiro-Coimbra: 10.0.1.32/30
- DC.C1-Coimbra: 10.0.1.36/30
```

## Porto Router

```
configure terminal

mpls ip

interface f0/0
ip address 10.0.1.2 255.255.255.252
ip ospf 1 area 0
no shutdown
mpls ip


interface f0/1
ip address 10.0.1.6 255.255.255.252
ip ospf 1 area 0
no shutdown
mpls ip

interface f1/0
ip address 10.0.1.9 255.255.255.252
ip ospf 1 area 0
no shutdown
mpls ip

interface f1/1
ip address 10.0.1.13 255.255.255.252
ip ospf 1 area 0
no shutdown
mpls ip
```

## Aveiro Router

```
configure terminal

mpls ip

interface f0/0
ip address 10.0.1.25 255.255.255.252
ip ospf 1 area 0
no shutdown
mpls ip

interface f0/1
ip address 10.0.1.29 255.255.255.252
ip ospf 1 area 0
no shutdown
mpls ip

interface f1/0
ip address 10.0.1.10 255.255.255.252
ip ospf 1 area 0
no shutdown
mpls ip

interface f1/1
ip address 10.0.1.33 255.255.255.252
ip ospf 1 area 0
no shutdown
mpls ip
```

## Coimbra Router

```
configure terminal

mpls ip

interface f0/0
ip address 10.0.1.37 255.255.255.252
ip ospf 1 area 0
no shutdown
mpls ip

interface f0/1
ip address 10.0.1.34 255.255.255.252
ip ospf 1 area 0
no shutdown
mpls ip

interface f1/0
ip address 10.0.1.14 255.255.255.252
ip ospf 1 area 0
no shutdown
mpls ip

interface f1/1
ip address 10.0.1.21 255.255.255.252
ip ospf 1 area 0
no shutdown
mpls ip
```

## Lisboa Router

```
configure terminal

mpls ip

interface f0/0
ip address 10.0.1.2 255.255.255.252
ip ospf 1 area 0
no shutdown
mpls ip

interface f0/1
ip address 10.0.1.6 255.255.255.252
ip ospf 1 area 0
no shutdown
mpls ip
```

## DC.P1

```
configure terminal

interface Lo0
ip address 10.0.0.1 255.255.255.255
ip ospf 1 area 0
no shutdown

interface f0/0
ip address 10.0.1.1 255.255.255.252
ip ospf 1 area 0
no shutdown

interface f0/1
ip address 10.0.2.2 255.255.255.0
no shutdown
```

### MPLS Layer 3 VPN for client Z2

```
configure terminal

ip vrf VPN-Z
rd 43100:1
route-target export 43100:1
route-target import 43100:1

interface FastEthernet0/1
ip vrf forwarding VPN-Z
ip address 10.0.2.2 255.255.255.0
```

### Enable MPLS (LDP)

```
configure terminal
mpls ip

interface f0/0
mpls ip

router bgp 43100
bgp router-id 10.0.0.1
neighbor 10.0.0.5 remote-as 43100
neighbor 10.0.0.5 update-source Loopback0
neighbor 10.0.0.6 remote-as 43100
neighbor 10.0.0.6 update-source Loopback0

address-family vpnv4
neighbor 10.0.0.5 activate
neighbor 10.0.0.5 send-community both
neighbor 10.0.0.6 activate
neighbor 10.0.0.6 send-community both

address-family ipv4 vrf VPN-Z
redistribute connected
```

## DC.P2

```
configure
set interfaces dummy dum0 address 10.0.0.2/32
set interfaces ethernet eth0 address 10.0.1.5/30
set protocols ospf area 0 network 10.0.1.4/30   
set protocols ospf area 0 network 10.0.0.2/32
set system host-name DC.P2
commit
save
```

### Layer 2 VPN for Client X2

```
configure

set protocols bgp system-as 43100
set protocols bgp address-family l2vpn-evpn advertise-all-vni
set protocols bgp parameters router-id 10.0.0.2
set protocols bgp neighbor 10.0.0.3 peer-group evpn
set protocols bgp neighbor 10.0.0.4 peer-group evpn
set protocols bgp peer-group evpn update-source dum0
set protocols bgp peer-group evpn remote-as 43100
set protocols bgp peer-group evpn address-family l2vpn-evpn nexthop-self
set protocols bgp peer-group evpn address-family l2vpn-evpn route-reflector-client

commit
save
```

### Configure VXLAN for Client X2

```
configure

set interfaces vxlan vxlan1000 source-address 10.0.0.2
set interfaces vxlan vxlan1000 vni 1000
set interfaces vxlan vxlan1000 mtu 1500

set interfaces bridge br1000 address 10.2.2.1/22
set interfaces bridge br1000 description 'Client X'
set interfaces bridge br1000 member interface eth2
set interfaces bridge br1000 member interface vxlan1000

commit
save
```

### Sub-interfaces for Client Y VLAN 20 and VLAN 30

```
configure
set interfaces ethernet eth1 vif 20
set interfaces ethernet eth1 vif 30
commit
save
```

### Configure VXLAN for client Y

```
configure

set interfaces ethernet eth1 vif 20
set interfaces ethernet eth1 vif 30

set interfaces vxlan vxlan1020 vni 1020
set interfaces vxlan vxlan1020 mtu 1500
set interface vxlan vxlan1020 remote 10.0.0.3
set interface vxlan vxlan1020 remote 10.0.0.4

set interfaces vxlan vxlan1030 vni 1030
set interfaces vxlan vxlan1030 mtu 1500
set interface vxlan vxlan1030 remote 10.0.0.3
set interface vxlan vxlan1030 remote 10.0.0.4

set interfaces bridge br1020 member interface 'eth1.20'
set interfaces bridge br1020 member interface 'vxlan1020'
set interfaces bridge br1030 member interface 'eth1.30'
set interfaces bridge br1030 member interface 'vxlan1030'

commit
save
```

## DC.A1

```
configure

set interfaces dummy dum0 address 10.0.0.4/32
set interfaces ethernet eth0 address 10.0.1.26/30
set protocols ospf area 0 network 10.0.1.24/30
set protocols ospf area 0 network 10.0.0.4/32
set system host-name DC.A1

commit
save
```

### Layer 2 VPN

```
configure

set protocols bgp system-as 43100
set protocols bgp address-family l2vpn-evpn advertise-all-vni
set protocols bgp parameters router-id 10.0.0.4
set protocols bgp neighbor 10.0.0.2 peer-group evpn
set protocols bgp neighbor 10.0.0.3 peer-group evpn
set protocols bgp peer-group evpn update-source dum0
set protocols bgp peer-group evpn remote-as 43100
set protocols bgp peer-group evpn address-family l2vpn-evpn nexthop-self

commit
save
```

### Configure VXLAN

```
configure

set interfaces vxlan vxlan1000 source-address 10.0.0.4
set interfaces vxlan vxlan1000 vni 1000
set interfaces vxlan vxlan1000 mtu 1500

set interfaces bridge br1000 address 10.2.3.1/22
set interfaces bridge br1000 description 'Client X'
set interfaces bridge br1000 member interface eth2
set interfaces bridge br1000 member interface vxlan1000

commit
save
```

### Sub-interfaces for Client Y VLAN 20 and VLAN 30

```
configure
set interfaces ethernet eth1 vif 20
set interfaces ethernet eth1 vif 30
commit
save
```

### Configure VXLAN for client Y

```
configure

set interfaces vxlan vxlan1020 vni 1020
set interfaces vxlan vxlan1020 mtu 1500
set interface vxlan vxlan1020 remote 10.0.0.2
set interface vxlan vxlan1020 remote 10.0.0.3

set interfaces vxlan vxlan1030 vni 1030
set interfaces vxlan vxlan1030 mtu 1500
set interface vxlan vxlan1030 remote 10.0.0.2
set interface vxlan vxlan1030 remote 10.0.0.3

set interfaces bridge br1020 member interface 'eth1.20'
set interfaces bridge br1020 member interface 'vxlan1020'
set interfaces bridge br1030 member interface 'eth1.30'
set interfaces bridge br1030 member interface 'vxlan1030'

commit
save
```

## DC.A2

```
configure terminal

interface Lo0
ip address 10.0.0.5 255.255.255.255
ip ospf 1 area 0
no shutdown

interface f0/0
ip address 10.0.1.30 255.255.255.252
ip ospf 1 area 0
no shutdown

interface f0/1
ip address 10.0.3.3 255.255.255.0
no shutdown
```

### MPLS Layer 3 VPN for client Z3

```
configure terminal

ip vrf VPN-Z
rd 43100:1
route-target export 43100:1
route-target import 43100:1

interface FastEthernet0/1
ip vrf forwarding VPN-Z
ip address 10.0.3.3 255.255.255.0
```

### Enable MPLS (LDP)

```
configure terminal
mpls ip

interface f0/0
mpls ip

router bgp 43100
bgp router-id 10.0.0.5
neighbor 10.0.0.1 remote-as 43100
neighbor 10.0.0.1 update-source Loopback0
neighbor 10.0.0.6 remote-as 43100
neighbor 10.0.0.6 update-source Loopback0

address-family vpnv4
neighbor 10.0.0.1 activate
neighbor 10.0.0.1 send-community both
neighbor 10.0.0.6 activate
neighbor 10.0.0.6 send-community both

address-family ipv4 vrf VPN-Z
redistribute connected
```