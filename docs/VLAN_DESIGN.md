# VLAN Design

## Overview

The enterprise network uses VLAN segmentation to separate users, management traffic, server infrastructure, voice traffic, native trunk traffic, and unused switchports.

The design uses two multilayer switches as redundant Layer 3 gateways and four access switches connected redundantly to both multilayer switches.

## VLAN Allocation

| VLAN | Name       | Purpose                            | IPv4 Network    | IPv6 Network         |
| ---: | ---------- | ---------------------------------- | --------------- | -------------------- |
|   10 | IT         | IT/user endpoints                  | 192.168.10.0/24 | 2001:DB8:100:10::/64 |
|   20 | HR         | Human Resources endpoints          | 192.168.20.0/24 | 2001:DB8:100:20::/64 |
|   30 | SALES      | Sales endpoints                    | 192.168.30.0/24 | 2001:DB8:100:30::/64 |
|   40 | MANAGEMENT | Network/device management          | 192.168.40.0/24 | 2001:DB8:100:40::/64 |
|   50 | SERVERS    | Infrastructure/application servers | 192.168.50.0/24 | 2001:DB8:100:50::/64 |
|   60 | VOICE      | Voice/phone traffic                | 192.168.60.0/24 | 2001:DB8:100:60::/64 |
|   99 | NATIVE     | Native VLAN for trunks             | —               | —                    |
|  999 | UNUSED     | Unused/native-isolation VLAN       | —               | —                    |

## VLAN 10 — IT

VLAN 10 is assigned to IT endpoints.

```text
IPv4:
192.168.10.0/24

HSRP Gateway:
192.168.10.1

MLS1:
192.168.10.2

MLS2:
192.168.10.3

IPv6:
2001:DB8:100:10::/64

MLS1:
2001:DB8:100:10::2

MLS2:
2001:DB8:100:10::3
```

PC1 receives its IPv4 address through DHCP and its IPv6 address through SLAAC.

## VLAN 20 — HR

VLAN 20 is assigned to HR endpoints.

```text
IPv4:
192.168.20.0/24

HSRP Gateway:
192.168.20.1

MLS1:
192.168.20.2

MLS2:
192.168.20.3

IPv6:
2001:DB8:100:20::/64
```

PC2 receives its IPv4 address through DHCP and its IPv6 configuration through SLAAC.

An edge ACL prevents HR traffic from reaching the simulated Internet.

## VLAN 30 — SALES

VLAN 30 is assigned to Sales endpoints.

```text
IPv4:
192.168.30.0/24

HSRP Gateway:
192.168.30.1

IPv6:
2001:DB8:100:30::/64
```

PC3 receives IPv4 through DHCP and IPv6 through SLAAC.

## VLAN 40 — MANAGEMENT

VLAN 40 is dedicated to network-device management.

```text
IPv4:
192.168.40.0/24

HSRP Gateway:
192.168.40.1

IPv6:
2001:DB8:100:40::/64
```

Static management addresses:

```text
SW3 → 192.168.40.11
SW4 → 192.168.40.12
SW5 → 192.168.40.13
SW6 → 192.168.40.14
```

PC4 and PC8 operate as management/testing endpoints.

SSH is enabled on the access switches so administrators can manage them remotely.

## VLAN 50 — SERVERS

VLAN 50 contains infrastructure and application servers.

```text
IPv4:
192.168.50.0/24

HSRP Gateway:
192.168.50.1

IPv6:
2001:DB8:100:50::/64
```

### Server1

```text
IPv4 → 192.168.50.10
IPv6 → 2001:DB8:100:50::10
```

Primary services:

* DHCP

### Server2

```text
IPv4 → 192.168.50.11
IPv6 → 2001:DB8:100:50::11
```

Primary services:

* DNS
* HTTP/HTTPS
* NTP
* Syslog

Server2 is also used as the central management-services endpoint for the network.

## VLAN 60 — VOICE

VLAN 60 is reserved for voice traffic.

```text
IPv4:
192.168.60.0/24

HSRP Gateway:
192.168.60.1

IPv6:
2001:DB8:100:60::/64
```

The VLAN was configured and prepared for DHCP and IPv6 addressing.

No physical IP phones were included in the final topology, so existing PC access ports were not converted into combined data/voice ports.

## VLAN 99 — Native VLAN

VLAN 99 is used as the native VLAN on 802.1Q trunks.

The native VLAN is separated from the user VLANs to avoid using a production user VLAN for untagged trunk traffic.

## VLAN 999 — Unused VLAN

VLAN 999 is reserved for unused switchports.

Unused interfaces were placed into the unused VLAN and administratively disabled as part of the access-layer hardening.

## Trunk Design

Each access switch has redundant uplinks to both multilayer switches.

Example:

```text
SW3 Fa0/23 → MLS1
SW3 Fa0/24 → MLS2
```

The same redundant-uplink design is used on SW4, SW5, and SW6.

802.1Q trunking carries the required VLANs between the access and distribution/core layers.

The native VLAN is:

```text
VLAN 99
```

## EtherChannel

LACP EtherChannel was implemented on access-switch uplinks.

Example:

```text
Port-channel 1

Fa0/23
Fa0/24
```

This provides link aggregation and improves availability if an individual member link fails.

## Spanning Tree

Rapid PVST+ is used for Layer 2 loop prevention.

The redundant topology intentionally creates multiple physical paths, so STP is required to prevent Layer 2 loops while still allowing redundant connectivity.

## Security Considerations

The VLAN design is combined with:

* Port Security
* DHCP Snooping
* Dynamic ARP Inspection
* SSH management
* Management VLAN separation
* Unused VLAN isolation
* IPv4/IPv6 ACL policies

## Design Rationale

The VLAN structure separates user roles and infrastructure functions while allowing centralized routing at the multilayer-switch layer.

This makes it possible to apply routing and security policy by VLAN instead of treating the entire LAN as a single broadcast domain.

## Final VLAN Architecture

```text
                    MLS1 ===== MLS2
                    /  \       /  \
                   /    \     /    \
                 SW3    SW4  SW5    SW6
                  │      │    │      │
                 VLANs  VLANs VLANs  VLANs

10  IT
20  HR
30  SALES
40  MANAGEMENT
50  SERVERS
60  VOICE
99  NATIVE
999 UNUSED
```
