# Network Test Results

## 1. DHCP

### VLAN 10

PC1 successfully received:

```text
192.168.10.101
```

### VLAN 20

PC2 successfully received:

```text
192.168.20.100
```

VLANs 30, 40, and 60 were also configured and validated using DHCP.

## 2. DNS

Internal DNS resolution was tested using:

```text
server1.enterprise.local
server2.enterprise.local
www.enterprise.local
```

Both IPv4 and IPv6 records were configured.

## 3. OSPF

All expected IPv4 OSPF adjacencies reached:

```text
FULL
```

R1 demonstrated multiple equal-cost paths toward internal VLAN networks.

## 4. OSPFv3

All expected IPv6 OSPFv3 adjacencies reached:

```text
FULL
```

The IPv6 VLAN networks and router loopbacks were successfully exchanged.

## 5. NAT/PAT

Internal IPv4 clients successfully reached the simulated Internet destination:

```text
8.8.8.8
```

NAT translations were observed on the edge routers.

## 6. IPv6 Internet Connectivity

Internal IPv6 clients successfully reached:

```text
2001:DB8:8888::8
```

The test required both the forward path and the ISP-side return route to the enterprise IPv6 prefix.

## 7. SSH

SSH version 2 was successfully tested against:

* MLS1
* MLS2
* R1
* R2
* SW3
* SW4
* SW5
* SW6

## 8. Failover Testing

### IPv4 OSPF Failover

A routed R1-to-MLS1 link was shut down.

Result:

```text
Alternate OSPF path remained available.
Connectivity continued.
```

### IPv6 OSPFv3 Failover

An IPv6 transit link was shut down.

Result:

```text
OSPFv3 reconverged.
IPv6 connectivity continued through the alternate path.
```

### WAN Failover

R1's WAN interface was shut down.

Result:

```text
Internal clients continued reaching the simulated Internet through R2.
```

### HSRP Failover

MLS1's HSRP priority was temporarily reduced.

Result:

```text
MLS2 became Active.
Connectivity continued.
MLS1 was restored and reclaimed Active status.
```

## 9. Security Validation

The following were verified:

```text
Port Security
DHCP Snooping
Dynamic ARP Inspection
SSH
IPv4 ACL
IPv6 ACL
```

The HR Internet restriction was tested against both IPv4 and IPv6 simulated Internet destinations.

## 10. Services Validation

The following services were successfully configured and tested:

```text
DHCP
DNS
HTTP/HTTPS
NTP
Syslog
```

## Final Result

The completed topology successfully demonstrated:

```text
Layer 2 redundancy
Layer 3 redundancy
IPv4 dynamic routing
IPv6 dynamic routing
WAN redundancy
Gateway redundancy
Network security
Centralized services
Failure detection and recovery
```
