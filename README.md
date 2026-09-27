# Enterprise Dual-Stack Network Infrastructure & High-Availability Lab

![Enterprise Dual-Stack Network Topology](screenshots/01-topology.png)

## Overview

A complete enterprise network infrastructure lab designed and implemented in Cisco Packet Tracer.

The project demonstrates the design, configuration, security, services, redundancy, and troubleshooting of a dual-stack IPv4/IPv6 enterprise network.

The environment was built from the ground up and tested through multiple simulated failure scenarios to validate routing, gateway redundancy, WAN redundancy, and end-to-end connectivity.

## Objectives

* Design a scalable enterprise network topology
* Implement VLAN segmentation and Layer 2 redundancy
* Implement inter-VLAN routing using multilayer switches
* Provide redundant default gateways using HSRP/HSRPv6
* Implement dynamic routing with OSPF and OSPFv3
* Provide centralized network services
* Secure the access layer
* Implement IPv4 and IPv6 ACL policies
* Implement NAT/PAT and dual-WAN connectivity
* Validate network resilience through controlled failure testing
* Practice enterprise network administration using Cisco IOS

## Topology

### Core Architecture

```text
                         ┌──────────────┐
                         │    ISP1      │
                         └──────┬─┬─────┘
                                │ │
                         ┌──────┘ └──────┐
                         │               │
                        R1               R2
                       /  \             /  \
                      /    \           /    \
                   MLS1====================MLS2
                  / │  │  │              │  │ \
                 /  │  │  │              │  │  \
               SW3 SW4 SW5 SW6          Access Layer

                         VLAN 50
                      ┌──────┴──────┐
                      │             │
                   Server1       Server2
```

The design intentionally avoids a direct R1-to-R2 connection. Each edge router has independent connections to both multilayer switches, providing multiple paths through the enterprise core.

## VLAN Design

| VLAN | Name       | IPv4 Network    | IPv6 Network         |
| ---: | ---------- | --------------- | -------------------- |
|   10 | IT         | 192.168.10.0/24 | 2001:DB8:100:10::/64 |
|   20 | HR         | 192.168.20.0/24 | 2001:DB8:100:20::/64 |
|   30 | SALES      | 192.168.30.0/24 | 2001:DB8:100:30::/64 |
|   40 | MANAGEMENT | 192.168.40.0/24 | 2001:DB8:100:40::/64 |
|   50 | SERVERS    | 192.168.50.0/24 | 2001:DB8:100:50::/64 |
|   60 | VOICE      | 192.168.60.0/24 | 2001:DB8:100:60::/64 |
|   99 | NATIVE     | —               | —                    |
|  999 | UNUSED     | —               | —                    |

## IPv4 Addressing

### HSRP Gateways

```text
VLAN 10 → 192.168.10.1
VLAN 20 → 192.168.20.1
VLAN 30 → 192.168.30.1
VLAN 40 → 192.168.40.1
VLAN 50 → 192.168.50.1
VLAN 60 → 192.168.60.1
```

### Server Addresses

```text
Server1 → 192.168.50.10
Server2 → 192.168.50.11
```

### OSPF Transit Networks

```text
R1 ↔ MLS1 → 10.0.1.0/30
R1 ↔ MLS2 → 10.0.2.0/30
R2 ↔ MLS1 → 10.0.3.0/30
R2 ↔ MLS2 → 10.0.4.0/30
```

### IPv4 WAN

```text
R1 ↔ ISP1 → 203.0.113.0/30
R2 ↔ ISP1 → 198.51.100.0/30
```

The WAN addresses use documentation-only TEST-NET address space for simulation.

## IPv6 Addressing

### OSPFv3 Transit Networks

```text
R1 ↔ MLS1 → 2001:DB8:1000:1::/64
R1 ↔ MLS2 → 2001:DB8:1000:2::/64
R2 ↔ MLS1 → 2001:DB8:1000:3::/64
R2 ↔ MLS2 → 2001:DB8:1000:4::/64
```

### Router Loopbacks

```text
R1   → 2001:DB8:FFFF::1/128
R2   → 2001:DB8:FFFF::2/128
MLS1 → 2001:DB8:FFFF::3/128
MLS2 → 2001:DB8:FFFF::4/128
```

### Server IPv6

```text
Server1 → 2001:DB8:100:50::10/64
Server2 → 2001:DB8:100:50::11/64
```

### IPv6 WAN

```text
R1 ↔ ISP1 → 2001:DB8:2000:1::/64
R2 ↔ ISP1 → 2001:DB8:2000:2::/64
```

Simulated external IPv6 destination:

```text
2001:DB8:8888::8/128
```

## Layer 2 Technologies

### VLAN Segmentation

Separate VLANs were created for:

* IT
* HR
* Sales
* Management
* Servers
* Voice
* Native traffic
* Unused ports

### Trunking

802.1Q trunks connect the access layer to both multilayer switches.

Native VLAN:

```text
VLAN 99
```

Unused VLAN:

```text
VLAN 999
```

### EtherChannel

LACP EtherChannel was configured on the access-switch uplinks.

Example:

```text
Port-channel 1
FastEthernet0/23
FastEthernet0/24
```

### Spanning Tree

Rapid PVST+ was used to provide Layer 2 loop prevention and redundancy.

## Layer 3 Technologies

### Inter-VLAN Routing

MLS1 and MLS2 provide gateway SVIs for all production VLANs.

### HSRP / HSRPv6

HSRP provides IPv4 first-hop redundancy.

HSRPv6 provides IPv6 first-hop redundancy.

MLS1 was configured as the preferred active gateway, with MLS2 operating as standby under normal conditions.

### OSPF

OSPF Area 0 provides dynamic IPv4 routing between R1, R2, MLS1, and MLS2.

Equal-cost paths are available through the redundant multilayer-switch paths.

### OSPFv3

OSPFv3 provides dynamic IPv6 routing across the same redundant core topology.

## DHCP

Server1 provides centralized DHCP services.

DHCP relay was configured on the multilayer-switch SVIs so clients in different VLANs can obtain addresses from the server VLAN.

Configured DHCP scopes include:

```text
VLAN 10
VLAN 20
VLAN 30
VLAN 40
VLAN 60
```

## DNS

Server2 provides internal DNS services.

Configured records include:

```text
server1.enterprise.local
server2.enterprise.local
www.enterprise.local
```

Both IPv4 A records and IPv6 AAAA records were configured.

## NTP

Server2 provides the centralized NTP source:

```text
192.168.50.11
```

Network devices were configured as NTP clients to maintain consistent timestamps.

## Syslog

Server2 acts as the centralized Syslog server.

Network devices send informational-level logs to:

```text
192.168.50.11
```

This provides a centralized location for network event monitoring.

## NAT/PAT

R1 and R2 provide NAT/PAT toward the WAN.

Internal private IPv4 networks are translated using the respective WAN interface addresses.

```text
Inside:
192.168.0.0/16

R1 Outside:
203.0.113.2

R2 Outside:
198.51.100.2
```

## WAN Redundancy

Both R1 and R2 have independent WAN connections to ISP1.

Default IPv4 and IPv6 routes point toward ISP1.

R1 and R2 advertise their default routes into OSPF/OSPFv3 so the internal network has redundant paths to the simulated Internet.

## Security

### Port Security

Port security was implemented on access ports to restrict unauthorized endpoint access.

### DHCP Snooping

DHCP Snooping was enabled on client VLANs.

Trusted interfaces:

```text
Fa0/23
Fa0/24
```

Client access ports remain untrusted.

### Dynamic ARP Inspection

DAI was enabled on client VLANs and integrated with DHCP Snooping bindings.

### SSH

SSH version 2 was configured on the network infrastructure.

Management access uses:

```text
Local authentication
SSH only
RSA keys
```

Telnet access was removed from the VTY configuration.

### IPv4 ACL

An edge ACL prevents HR traffic from reaching the simulated IPv4 Internet destination.

Policy:

```text
192.168.20.0/24 → 8.8.8.8
DENY
```

Other traffic is permitted.

### IPv6 ACL

A corresponding IPv6 edge ACL prevents HR traffic from reaching the simulated IPv6 Internet destination.

Policy:

```text
2001:DB8:100:20::/64
        ↓
2001:DB8:8888::8
        ↓
DENY
```

Other traffic is permitted.

## IPv6 Client Configuration

Client VLANs use SLAAC for IPv6 addressing.

Example VLAN 10 client:

```text
2001:DB8:100:10::/64
```

IPv6 gateway redundancy is provided using HSRPv6.

## Services

| Service    | Device  | Address       |
| ---------- | ------- | ------------- |
| DHCP       | Server1 | 192.168.50.10 |
| DNS        | Server2 | 192.168.50.11 |
| HTTP/HTTPS | Server2 | 192.168.50.11 |
| NTP        | Server2 | 192.168.50.11 |
| Syslog     | Server2 | 192.168.50.11 |

## Resilience Testing

The network was tested under simulated failures.

### IPv4 routing failover

An R1-to-MLS1 routed link was shut down.

OSPF reconverged and traffic continued through MLS2.

### IPv6 routing failover

An IPv6 R1-to-MLS1 transit link was shut down.

OSPFv3 reconverged and IPv6 traffic continued through the alternate path.

### WAN failover

R1's WAN interface was temporarily shut down.

Internal clients continued reaching the simulated Internet through R2.

### HSRP failover

MLS1's HSRP priority was temporarily reduced.

MLS2 became active, after which MLS1 was restored and reclaimed the active role through preemption.

## Verification

The final lab was verified using commands including:

```text
show ip route
show ipv6 route
show ip ospf neighbor
show ipv6 ospf neighbor
show standby brief
show etherchannel summary
show interfaces trunk
show ip nat translations
show ip dhcp snooping binding
show ip arp inspection
show ip ssh
show logging
show ntp status
```

End-to-end connectivity was tested using IPv4, IPv6, DNS, HTTP/HTTPS, SSH, and simulated Internet destinations.

## Known Limitation

IP Source Guard was not implemented because the Cisco Packet Tracer 2960 IOS image used for this lab did not expose the required `ip verify source` command.

The feature was therefore not represented as completed.

## Key Learning Outcomes

This project provided practical experience with:

* Enterprise network design
* Cisco IOS configuration
* VLAN segmentation
* Layer 2 redundancy
* Layer 3 routing
* OSPF and OSPFv3
* IPv4 and IPv6
* HSRP and HSRPv6
* DHCP and DHCP relay
* DNS
* NAT/PAT
* ACLs
* Network security
* SSH
* NTP
* Syslog
* Network troubleshooting
* High-availability design
* Failure and recovery testing

## Project Status

**Completed**

Cisco Packet Tracer enterprise dual-stack infrastructure lab.

Built, configured, tested, and documented from the ground up.
