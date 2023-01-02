# Porto

```
configure terminal

mpls ip

interface f0/0
ip address 10.0.1.2 255.255.255.252
ip ospf 1 area 0
no shutdown
mpls ip

interface f0/1
ip address 10.0.1.5 255.255.255.252
ip ospf 1 area 0
no shutdown
mpls ip
```

# Aveiro

```
configure terminal

mpls ip

interface f0/0
ip address 10.0.1.6 255.255.255.252
ip ospf 1 area 0
no shutdown
mpls ip

interface f0/1
ip address 10.0.1.9 255.255.255.252
ip ospf 1 area 0
no shutdown
mpls ip
```

# DCP1

```
configure terminal

mpls ip

interface Loopback0
ip address 10.0.0.1 255.255.255.255
ip ospf 1 area 0
no shutdown

interface f0/0
ip address 10.1.0.1 255.255.255.0
no shutdown

interface f0/1
ip address 10.0.1.1 255.255.255.252
ip ospf 1 area 0
no shutdown
mpls ip

ip vrf VPN-Z
rd 43100:1
route-target export 43100:1
route-target import 43100:1

interface FastEthernet0/0
ip vrf forwarding VPN-Z
ip address 10.1.0.1 255.255.255.0

router bgp 43100
bgp router-id 10.0.0.1
neighbor 10.0.0.2 remote-as 43100
neighbor 10.0.0.2 update-source Loopback0

address-family vpnv4
neighbor 10.0.0.2 activate
neighbor 10.0.0.2 send-community both

address-family ipv4 vrf VPN-Z
redistribute connected
```

# DCA2

```
configure terminal

mpls ip

interface Loopback0
ip address 10.0.0.2 255.255.255.255
ip ospf 1 area 0
no shutdown

interface f0/0
ip address 10.2.0.1 255.255.255.0
no shutdown

interface f0/1
ip address 10.0.1.10 255.255.255.252
ip ospf 1 area 0
no shutdown
mpls ip

ip vrf VPN-Z
rd 43100:1
route-target export 43100:1
route-target import 43100:1

interface FastEthernet0/0
ip vrf forwarding VPN-Z
ip address 10.2.0.1 255.255.255.0

router bgp 43100
bgp router-id 10.0.0.2
neighbor 10.0.0.1 remote-as 43100
neighbor 10.0.0.1 update-source Loopback0

address-family vpnv4
neighbor 10.0.0.1 activate
neighbor 10.0.0.1 send-community both

address-family ipv4 vrf VPN-Z
redistribute connected
```