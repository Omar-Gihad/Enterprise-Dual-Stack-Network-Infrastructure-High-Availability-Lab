# Routing Design

## Overview

The network uses a hierarchical Layer 3 design centered around two multilayer switches and two edge routers.

The routing architecture combines:

* Inter-VLAN routing
* HSRP
* OSPF
* OSPFv3
* Static default routes
* Dual-WAN connectivity
* NAT/PAT
* IPv4 and IPv6 default-route propagation

The design intentionally does not include a direct R1-to-R2 physical connection.

## Layer 3 Architecture

```text
                         ISP1
                       /      \
                      R1      R2
                     /  \    /  \
                    /    \  /    \
                  MLS1 ======== MLS2
                   |              |
                Access Layer   Access Layer
```

R1 and R2 each connect to both multilayer switches.

This creates four independent internal routed paths:

```text
R1 ↔ MLS1
R1 ↔ MLS2
R2 ↔ MLS1
R2 ↔ MLS2
```

## Inter-VLAN Routing

MLS1 and MLS2 provide Layer 3 SVIs for the production VLANs.

Each VLAN has an individual IPv4 and IPv6 gateway on both multilayer switches.

Example:

```text
VLAN 10

MLS1 → 192.168.10.2
MLS2 → 192.168.10.3
HSRP → 192.168.10.1
```

IPv6:

```text
MLS1 → 2001:DB8:100:10::2
MLS2 → 2001:DB8:100:10::3
```

## HSRP

HSRP provides redundant IPv4 default gateways.

Under normal operation:

```text
MLS1 → Active
MLS2 → Standby
```

The same design is used across the production VLANs.

HSRP priority was set so MLS1 is preferred, while `preempt` allows it to reclaim the active role after recovery.

## HSRPv6

HSRPv6 provides first-hop redundancy for IPv6.

The lab uses automatically generated HSRPv6 virtual link-local addresses with:

```text
standby <group> ipv6 autoconfig
```

This approach was used because it is supported by the Catalyst 3560 Packet Tracer environment used for the lab.

## IPv4 OSPF

OSPF Area 0 is used for internal IPv4 dynamic routing.

OSPF-enabled devices:

```text
R1
R2
MLS1
MLS2
```

### IPv4 Transit Networks

```text
R1 ↔ MLS1 → 10.0.1.0/30
R1 ↔ MLS2 → 10.0.2.0/30
R2 ↔ MLS1 → 10.0.3.0/30
R2 ↔ MLS2 → 10.0.4.0/30
```

### Loopbacks

```text
R1   → 1.1.1.1/32
R2   → 2.2.2.2/32
MLS1 → 3.3.3.3/32
MLS2 → 4.4.4.4/32
```

Loopbacks provide stable OSPF router IDs and predictable endpoints for testing.

## OSPF ECMP

Because both MLS switches advertise the same internal VLAN networks, R1 and R2 can install equal-cost routes.

Example:

```text
192.168.10.0/24

via MLS1
via MLS2
```

This demonstrates OSPF Equal-Cost Multi-Path routing.

## IPv6 Routing

IPv6 uses OSPFv3 independently from IPv4 OSPF.

OSPFv3-enabled devices:

```text
R1
R2
MLS1
MLS2
```

## IPv6 Transit Networks

```text
R1 ↔ MLS1 → 2001:DB8:1000:1::/64
R1 ↔ MLS2 → 2001:DB8:1000:2::/64
R2 ↔ MLS1 → 2001:DB8:1000:3::/64
R2 ↔ MLS2 → 2001:DB8:1000:4::/64
```

## IPv6 Loopbacks

```text
R1   → 2001:DB8:FFFF::1/128
R2   → 2001:DB8:FFFF::2/128
MLS1 → 2001:DB8:FFFF::3/128
MLS2 → 2001:DB8:FFFF::4/128
```

## OSPFv3 Passive Interfaces

The user VLAN SVIs were initially capable of forming unintended OSPFv3 adjacencies between MLS1 and MLS2.

This was corrected by making the VLAN SVIs passive while keeping OSPFv3 enabled so their IPv6 prefixes could still be advertised.

The intended OSPFv3 adjacencies are only across the routed transit links:

```text
R1 ↔ MLS1
R1 ↔ MLS2
R2 ↔ MLS1
R2 ↔ MLS2
```

This keeps user VLANs as routed networks rather than OSPF neighbor segments.

## WAN Routing

R1 and R2 have independent WAN connections to ISP1.

### IPv4 WAN

```text
R1 ↔ ISP1
203.0.113.0/30

R2 ↔ ISP1
198.51.100.0/30
```

### IPv6 WAN

```text
R1 ↔ ISP1
2001:DB8:2000:1::/64

R2 ↔ ISP1
2001:DB8:2000:2::/64
```

Documentation-only address space is used for the simulated WAN.

## Default Routing

R1 and R2 each have their own default route toward ISP1.

IPv4:

```text
R1 → 203.0.113.1
R2 → 198.51.100.1
```

IPv6:

```text
R1 → 2001:DB8:2000:1::2
R2 → 2001:DB8:2000:2::2
```

## Default Route Propagation

R1 and R2 advertise their default routes into the internal routing domains.

IPv4 uses:

```text
default-information originate
```

under OSPF.

IPv6 uses the corresponding OSPFv3 default-information mechanism.

This allows the multilayer switches to learn external connectivity through the edge routers.

## Simulated Internet

ISP1 provides simulated external destinations:

```text
IPv4:
8.8.8.8/32

IPv6:
2001:DB8:8888::8/128
```

These addresses are used only inside the Packet Tracer simulation.

## IPv4 NAT/PAT

R1 and R2 perform PAT for internal private IPv4 networks.

Internal networks are summarized by:

```text
192.168.0.0/16
```

The WAN-facing interfaces operate as NAT outside interfaces, while the internal routed interfaces operate as NAT inside interfaces.

PAT allows multiple internal hosts to share the router's WAN address.

## IPv6 Addressing Model

IPv6 does not use NAT in this lab.

Internal IPv6 addresses are routed natively from the VLANs through the enterprise routing domain toward the simulated WAN.

This allows direct testing of:

```text
Client
→ IPv6 gateway
→ OSPFv3
→ Edge router
→ IPv6 WAN
→ ISP1
```

## Routing Failover

The design was tested under controlled failures.

### IPv4 Internal Failover

An R1-to-MLS1 routed link was shut down.

OSPF detected the failure and traffic continued through MLS2.

### IPv6 Internal Failover

An IPv6 R1-to-MLS1 transit path was shut down.

OSPFv3 reconverged and IPv6 connectivity continued through the alternate path.

### WAN Failover

R1's WAN interface was shut down.

Internal clients continued to reach the simulated Internet through R2.

## Routing Verification Commands

### IPv4

```text
show ip route
show ip route ospf
show ip ospf neighbor
show ip ospf interface brief
```

### IPv6

```text
show ipv6 route
show ipv6 route ospf
show ipv6 ospf neighbor
show ipv6 ospf interface brief
```

### HSRP

```text
show standby brief
show standby vlan 10
```

## Design Goals Achieved

The routing architecture provides:

* Dynamic IPv4 routing
* Dynamic IPv6 routing
* Multiple equal-cost paths
* Redundant first-hop gateways
* Independent WAN paths
* IPv4 NAT/PAT
* Native IPv6 routing
* Controlled convergence after failures
* End-to-end dual-stack connectivity
