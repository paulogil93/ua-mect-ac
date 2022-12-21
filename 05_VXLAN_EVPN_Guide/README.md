# VXLAN Guide

## Core Router

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

## PE1

```
configure
set interfaces ethernet eth0 address 192.0.1.1/24
set protocols ospf area 0 network 192.0.1.0/24
set system host-name PE1
commit
save
```

### Subinterfaces for VLAN 2 and VLAN 3

```
configure
set interfaces ethernet eth1 vif 2
set interfaces ethernet eth1 vif 3
commit
save
```

## PE2

```
configure
set interfaces ethernet eth0 address 192.0.2.2/24
set protocols ospf area 0 network 192.0.2.0/24
set system host-name PE2
commit
save
```

### Subinterfaces for VLAN 2 and VLAN 3

```
configure
set interfaces ethernet eth1 vif 2
set interfaces ethernet eth1 vif 3
commit
save
```