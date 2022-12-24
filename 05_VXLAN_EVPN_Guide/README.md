# VXLAN Guide

## Part 1 - VXLAN

### Core Router

```
configure terminal

interface f0/0
ip address 192.0.1.10 255.255.255.0
ip ospf 1 area 0
no shutdown

interface f0/1
ip address 192.0.2.10 255.255.255.0
ip ospf 1 area 0
no shutdown
```

### PE1

```
configure
set interfaces ethernet eth0 address 192.0.1.1/24
set protocols ospf area 0 network 192.0.1.0/24
set system host-name PE1
commit
save
```

#### Subinterfaces for VLAN 2 and VLAN 3

```
configure
set interfaces ethernet eth1 vif 2
set interfaces ethernet eth1 vif 3
commit
save
```

#### VXLAN connection between PE1 and PE2

```
configure
set interfaces vxlan vxlan102 vni 102
set interfaces vxlan vxlan102 mtu 1500
set interface vxlan vxlan102 remote 192.0.2.2
set interfaces vxlan vxlan103 vni 103
set interfaces vxlan vxlan103 mtu 1500
set interface vxlan vxlan103 remote 192.0.2.2
commit
```

#### Virtual bridges

> The virtual bridge interfaces can have IPv4 address and act as gateways of the VLAN.

```
configure
set interfaces bridge br102 member interface 'eth1.2'
set interfaces bridge br102 member interface 'vxlan102'
set interfaces bridge br103 member interface 'eth1.3'
set interfaces bridge br103 member interface 'vxlan103'
commit
```

### PE2

```
configure
set interfaces ethernet eth0 address 192.0.2.2/24
set protocols ospf area 0 network 192.0.2.0/24
set system host-name PE2
commit
save
```

#### Subinterfaces for VLAN 2 and VLAN 3

```
configure
set interfaces ethernet eth1 vif 2
set interfaces ethernet eth1 vif 3
commit
save
```

#### VXLAN connection between PE2 and PE1

```
configure
set interfaces vxlan vxlan102 vni 102
set interfaces vxlan vxlan102 mtu 1500
set interface vxlan vxlan102 remote 192.0.1.1
set interfaces vxlan vxlan103 vni 103
set interfaces vxlan vxlan103 mtu 1500
set interface vxlan vxlan103 remote 192.0.1.1
commit
```

#### Virtual bridges

> The virtual bridge interfaces can have IPv4 address and act as gateways of the VLAN.

```
configure
set interfaces bridge br102 member interface 'eth1.2'
set interfaces bridge br102 member interface 'vxlan102'
set interfaces bridge br103 member interface 'eth1.3'
set interfaces bridge br103 member interface 'vxlan103'
commit
```

## Part 2 - L2VPN/EVPN with VXLAN transport

### Core Router

```
configure terminal

interface f0/0
ip address 192.0.1.10 255.255.255.0
ip ospf 1 area 0
no shutdown

interface f0/1
ip address 192.0.2.10 255.255.255.0
ip ospf 1 area 0
no shutdown

interface f1/0
ip address 192.0.3.10 255.255.255.0
ip ospf 1 area 0
no shutdown
```

### PE1

```
configure
set interfaces dummy dum0 address 192.0.0.1/32
set interfaces ethernet eth0 address 192.0.1.1/24
set protocols ospf area 0 network 192.0.1.0/24
set protocols ospf area 0 network 192.0.0.0/24
set system host-name PE1
commit
save
```

#### Layer 2 VPN using a BGP EVPN with VXLAN transport

> PE1: Spine - Route Reflector

```
configure
set protocols bgp system-as 100
set protocols bgp address-family l2vpn-evpn advertise-all-vni
set protocols bgp parameters router-id 192.0.0.1
set protocols bgp neighbor 192.0.0.2 peer-group evpn
set protocols bgp neighbor 192.0.0.3 peer-group evpn
set protocols bgp peer-group evpn update-source dum0
set protocols bgp peer-group evpn remote-as 100
set protocols bgp peer-group evpn address-family l2vpn-evpn nexthop-self
set protocols bgp peer-group evpn address-family l2vpn-evpn route-reflector-client
commit
save
```

#### Configure VXLAN

```
configure

set interfaces vxlan vxlan101 source-address 192.0.0.1
set interfaces vxlan vxlan101 vni 101
set interfaces vxlan vxlan101 mtu 1500

set interfaces vxlan vxlan102 source-address 192.0.0.1
set interfaces vxlan vxlan102 vni 102
set interfaces vxlan vxlan102 mtu 1500

set interfaces bridge br101 address 10.1.1.1/16
set interfaces bridge br101 description 'customer blue'
set interfaces bridge br101 member interface eth1
set interfaces bridge br101 member interface vxlan101

set interfaces bridge br102 address 10.2.1.1/16
set interfaces bridge br102 description 'customer red'
set interfaces bridge br102 member interface eth2
set interfaces bridge br102 member interface vxlan102

commit
save
```

### PE2

```
configure
set interfaces dummy dum0 address 192.0.0.2/32
set interfaces ethernet eth0 address 192.0.2.2/24
set protocols ospf area 0 network 192.0.2.0/24
set protocols ospf area 0 network 192.0.0.0/24
set system host-name PE2
commit
save
```

#### Layer 2 VPN using a BGP EVPN with VXLAN transport

> PE2: Leaf - Route Reflector client

```
configure
set protocols bgp system-as 100
set protocols bgp address-family l2vpn-evpn advertise-all-vni
set protocols bgp parameters router-id 192.0.0.2
set protocols bgp neighbor 192.0.0.1 peer-group evpn
set protocols bgp peer-group evpn update-source dum0
set protocols bgp peer-group evpn remote-as 100
set protocols bgp peer-group evpn address-family l2vpn-evpn nexthop-self
commit
save
```

#### Configure VXLAN

```
configure

set interfaces vxlan vxlan101 source-address 192.0.0.2
set interfaces vxlan vxlan101 vni 101
set interfaces vxlan vxlan101 mtu 1500

set interfaces vxlan vxlan102 source-address 192.0.0.2
set interfaces vxlan vxlan102 vni 102
set interfaces vxlan vxlan102 mtu 1500

set interfaces bridge br101 address 10.1.2.1/16
set interfaces bridge br101 description 'customer blue'
set interfaces bridge br101 member interface eth1
set interfaces bridge br101 member interface vxlan101

set interfaces bridge br102 address 10.2.2.1/16
set interfaces bridge br102 description 'customer red'
set interfaces bridge br102 member interface eth2
set interfaces bridge br102 member interface vxlan102

commit
save
```

### PE3

```
configure
set interfaces dummy dum0 address 192.0.0.3/32
set interfaces ethernet eth0 address 192.0.3.3/24
set protocols ospf area 0 network 192.0.3.0/24
set protocols ospf area 0 network 192.0.0.0/24
set system host-name PE3
commit
save
```

#### Layer 2 VPN using a BGP EVPN with VXLAN transport

> PE3: Leaf - Route Reflector client

```
configure
set protocols bgp system-as 100
set protocols bgp address-family l2vpn-evpn advertise-all-vni
set protocols bgp parameters router-id 192.0.0.3
set protocols bgp neighbor 192.0.0.1 peer-group evpn
set protocols bgp peer-group evpn update-source dum0
set protocols bgp peer-group evpn remote-as 100
set protocols bgp peer-group evpn address-family l2vpn-evpn nexthop-self
commit
save
```

#### Configure VXLAN

```
configure

set interfaces vxlan vxlan101 source-address 192.0.0.3
set interfaces vxlan vxlan101 vni 101
set interfaces vxlan vxlan101 mtu 1500

set interfaces vxlan vxlan102 source-address 192.0.0.3
set interfaces vxlan vxlan102 vni 102
set interfaces vxlan vxlan102 mtu 1500

set interfaces bridge br101 address 10.1.3.1/16
set interfaces bridge br101 description 'customer blue'
set interfaces bridge br101 member interface eth1
set interfaces bridge br101 member interface vxlan101

set interfaces bridge br102 address 10.2.3.1/16
set interfaces bridge br102 description 'customer red'
set interfaces bridge br102 member interface eth2
set interfaces bridge br102 member interface vxlan102

commit
save
```