# Network Security Documentation

## Overview

Security was implemented across the access, management, routing, and WAN layers.

The project focuses on practical Layer 2 and Layer 3 controls rather than relying on a single security mechanism.

Implemented controls include:

```text
Port Security
DHCP Snooping
Dynamic ARP Inspection
SSH
IPv4 ACLs
IPv6 ACLs
Management VLAN
Unused VLAN isolation
NTP
Centralized Syslog
```

## Port Security

Port Security was configured on access ports on SW3–SW6.

The objective is to restrict unauthorized devices from using protected access interfaces.

Protected endpoint ports are associated with known endpoint access locations.

Verification:

```text
show port-security
show port-security interface fa0/1
show port-security interface fa0/2
```

Port Security was verified without reported violations during final testing.

## DHCP Snooping

DHCP Snooping was enabled on the client VLANs:

```text
10
20
30
40
60
```

Trusted interfaces are the redundant uplinks toward the multilayer switches:

```text
Fa0/23
Fa0/24
```

Client-facing ports remain untrusted.

A DHCP rate limit was configured on access ports:

```text
10 packets/second
```

### Security purpose

DHCP Snooping helps prevent unauthorized DHCP servers from responding to client requests and creates a binding database containing legitimate DHCP assignments.

Verification:

```text
show ip dhcp snooping
show ip dhcp snooping binding
```

The final lab contained valid DHCP bindings for the client endpoints.

## Dynamic ARP Inspection

Dynamic ARP Inspection was enabled on:

```text
VLAN 10
VLAN 20
VLAN 30
VLAN 40
VLAN 60
```

The uplinks are trusted:

```text
Fa0/23
Fa0/24
```

Client ports remain untrusted.

DAI uses DHCP Snooping information to validate ARP traffic.

Verification:

```text
show ip arp inspection
show ip arp inspection interfaces
```

## IP Source Guard

IP Source Guard was investigated but not included in the final implementation.

The Packet Tracer 2960 IOS image used in this project did not accept:

```text
ip verify source
```

Therefore, IP Source Guard was not represented as a completed feature.

This limitation was documented rather than bypassed with unsupported configuration.

## SSH Secure Management

SSH version 2 was configured across the infrastructure.

Configured devices include:

```text
R1
R2
MLS1
MLS2
SW3
SW4
SW5
SW6
```

The management architecture uses:

```text
Local username
+
Secret password
+
RSA keys
+
SSH version 2
+
VTY local authentication
+
SSH-only transport
```

Example management account:

```text
username admin secret class123
```

The credential is a lab-only value and should not be reused in production.

Verification:

```text
show ip ssh
show running-config | section line vty
```

## Management VLAN

VLAN 40 is dedicated to management traffic.

Static access-switch management addresses:

```text
SW3 → 192.168.40.11
SW4 → 192.168.40.12
SW5 → 192.168.40.13
SW6 → 192.168.40.14
```

Using a dedicated management VLAN separates administrative access from normal user traffic.

## IPv4 ACL Policy

An extended IPv4 ACL was implemented at the edge routers.

Policy:

```text
Source:
192.168.20.0/24

Destination:
8.8.8.8

Action:
DENY
```

Everything else is permitted.

The policy was applied inbound on the enterprise-facing interfaces of both R1 and R2.

This prevents HR traffic from bypassing the restriction through the alternate edge router.

Verification:

```text
show access-lists HR-INTERNET-BLOCK
```

The ACL counters were checked after generating test traffic.

## IPv6 ACL Policy

An IPv6 edge ACL implements the equivalent policy for the IPv6 simulated Internet.

Policy:

```text
Source:
2001:DB8:100:20::/64

Destination:
2001:DB8:8888::8/128

Action:
DENY
```

All other IPv6 traffic is permitted.

The ACL is applied to the enterprise-facing interfaces of both R1 and R2.

Verification:

```text
show ipv6 access-list HR-V6-INTERNET-BLOCK
```

## Defense in Depth

The security architecture combines controls at multiple layers.

```text
                        Enterprise Security
                               │
        ┌──────────────────────┼──────────────────────┐
        │                      │                      │
   Access Layer           Management Layer        Edge Layer
        │                      │                      │
 Port Security               SSH                  IPv4 ACL
 DHCP Snooping               NTP                  IPv6 ACL
 DAI                         Syslog               NAT/PAT
        │
   VLAN Isolation
```

This reduces dependence on any single security mechanism.

## NTP and Security Operations

Server2 provides the centralized NTP service:

```text
192.168.50.11
```

Network devices use Server2 as their NTP source.

Consistent time is important for interpreting logs and correlating events.

Verification:

```text
show ntp status
show ntp associations
```

## Centralized Syslog

Server2 also operates as the centralized Syslog server:

```text
192.168.50.11
```

Devices configured to send logs include:

```text
R1
R2
MLS1
MLS2
SW3
SW4
SW5
SW6
```

Informational-level messages are forwarded to the server.

Verification:

```text
show logging
```

A controlled configuration event was generated to confirm that logs reached Server2.

## Unused Ports

Unused interfaces were placed into VLAN 999 and administratively disabled.

The objective is to reduce the chance of unauthorized devices being connected to unused physical interfaces.

## Native VLAN

VLAN 99 was designated as the native VLAN for trunks.

Separating the native VLAN from production user VLANs avoids using a normal user VLAN for untagged trunk traffic.

## Secure Management Architecture

The intended administrative path is:

```text
Management PC
      │
   VLAN 40
      │
   Access Switch
      │
    SSHv2
      │
Network Device
```

Plain Telnet is not permitted on the configured VTY lines.

## Security Validation

The following security controls were verified during the project:

| Security Control       | Status                                     |
| ---------------------- | ------------------------------------------ |
| Port Security          | Implemented                                |
| DHCP Snooping          | Implemented                                |
| Dynamic ARP Inspection | Implemented                                |
| SSH v2                 | Implemented                                |
| IPv4 ACLs              | Implemented                                |
| IPv6 ACLs              | Implemented                                |
| Management VLAN        | Implemented                                |
| NTP                    | Implemented                                |
| Centralized Syslog     | Implemented                                |
| IP Source Guard        | Not implemented — Packet Tracer limitation |

## Testing Scenarios

### Unauthorized DHCP Protection

DHCP Snooping was configured with:

```text
Trusted:
Fa0/23–24

Untrusted:
Endpoint ports
```

This establishes the expected trust boundary.

### ARP Protection

DAI validates ARP traffic using DHCP Snooping bindings.

The resulting control chain is:

```text
DHCP Client
     ↓
DHCP Snooping Binding
     ↓
DAI validates ARP
```

### Unauthorized Internet Access

HR traffic was tested against both simulated Internet destinations.

IPv4:

```text
192.168.20.0/24
        ↓
8.8.8.8
```

IPv6:

```text
2001:DB8:100:20::/64
        ↓
2001:DB8:8888::8
```

Both were blocked by the corresponding edge ACL policies.

### Management Access

SSH access was successfully tested against the infrastructure devices using the dedicated management addressing.

## Security Design Principles Demonstrated

The project demonstrates:

* Segmentation
* Least-trust access boundaries
* Secure device management
* Control-plane redundancy
* Layer 2 protection
* Layer 3 filtering
* Centralized logging
* Centralized time synchronization
* Verification through controlled testing
* Explicit documentation of platform limitations
