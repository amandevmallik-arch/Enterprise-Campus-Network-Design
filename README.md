# Enterprise-Campus-Network-Design
````markdown
# Enterprise Campus Network Design — Cisco Packet Tracer

## Overview

This project simulates a secure and redundant enterprise campus network using Cisco Packet Tracer. The network is designed for a multi-department organization and includes VLAN segmentation, trunking, EtherChannel, STP/PVST optimization, inter-VLAN routing, HSRP gateway redundancy, OSPF dynamic routing, DHCP, NAT, ACL-based guest isolation, port security, and failover testing.

The goal of this project is to demonstrate practical enterprise networking skills aligned with CCNA-level concepts and real-world campus LAN operations.

---

## Project Objectives

- Design a scalable enterprise campus network with separate VLANs for different departments.
- Implement Layer 2 redundancy using STP/PVST and EtherChannel.
- Configure Layer 3 inter-VLAN routing using multilayer switches.
- Provide gateway redundancy using HSRP.
- Use OSPF area 0 for dynamic routing between the campus core and edge router.
- Configure DHCP for automatic IP address allocation.
- Enable internet access using NAT through an edge router.
- Restrict Guest VLAN access using ACLs.
- Secure access ports using port security.
- Validate network availability through HSRP, EtherChannel, and STP failover testing.

---

## Technologies Used

- Cisco Packet Tracer
- Cisco IOS CLI
- VLANs
- 802.1Q Trunking
- EtherChannel
- STP/PVST
- Inter-VLAN Routing
- HSRP
- OSPF Area 0
- DHCP
- NAT/PAT
- Access Control Lists
- Port Security

---

## Network Topology

The topology consists of:

- 1 Edge Router
- 1 ISP Router
- 2 Distribution Layer Multilayer Switches
- 2 Access Layer Switches
- Multiple end devices representing HR, Finance, IT, Guest, Management, and Server networks

Add topology screenshot here:

```markdown
![Network Topology](screenshots/topology.png)
````

---

## VLAN and IP Addressing Plan

| VLAN     | Department | Network          | Virtual Gateway |
| -------- | ---------- | ---------------- | --------------- |
| VLAN 10  | HR         | 192.168.10.0/24  | 192.168.10.1    |
| VLAN 20  | Finance    | 192.168.20.0/24  | 192.168.20.1    |
| VLAN 30  | IT         | 192.168.30.0/24  | 192.168.30.1    |
| VLAN 40  | Guest      | 192.168.40.0/24  | 192.168.40.1    |
| VLAN 99  | Management | 192.168.99.0/24  | 192.168.99.1    |
| VLAN 100 | Servers    | 192.168.100.0/24 | 192.168.100.1   |

---

## Key Features Implemented

### 1. VLAN Segmentation

Created 6 VLANs to separate department traffic and improve network organization, security, and scalability.

### 2. Trunking

Configured 802.1Q trunk links between distribution and access switches to carry multiple VLANs across the campus network.

### 3. EtherChannel

Implemented a 2-link EtherChannel between distribution switches to increase bandwidth and provide link-level redundancy.

### 4. STP/PVST Optimization

Configured STP root bridge priorities to optimize traffic paths and provide loop prevention across redundant Layer 2 links.

### 5. Inter-VLAN Routing

Configured SVIs on multilayer switches to enable communication between VLANs.

### 6. HSRP Gateway Redundancy

Implemented 6 HSRP virtual gateways to provide high availability for end-device default gateways.

### 7. OSPF Dynamic Routing

Configured OSPF area 0 between the edge router and distribution layer switches for dynamic route exchange.

### 8. DHCP Services

Configured 4 DHCP pools for automatic IP address assignment to HR, Finance, IT, and Guest VLANs.

### 9. NAT/PAT

Configured NAT overload on the edge router to allow internal VLAN users to access the simulated internet.

### 10. ACL-Based Guest Isolation

Applied an extended ACL to prevent Guest VLAN users from accessing internal enterprise networks while allowing internet access.

### 11. Port Security

Configured port security on access ports to restrict unauthorized device access and improve LAN security.

### 12. Failover Testing

Performed failover tests for HSRP, EtherChannel, and STP to validate network redundancy and high availability.

---

## Verification Commands

The following Cisco IOS commands were used to validate the configuration:

```bash
show vlan brief
show interfaces trunk
show etherchannel summary
show spanning-tree
show standby brief
show ip interface brief
show ip ospf neighbor
show ip route
show ip dhcp binding
show ip nat translations
show access-lists
show port-security
```

---

## Testing Performed

| Test Case                        | Expected Result                       | Status |
| -------------------------------- | ------------------------------------- | ------ |
| HR PC to Finance PC              | Successful inter-VLAN communication   | Passed |
| IT PC to HR PC                   | Successful inter-VLAN communication   | Passed |
| Guest PC to HR PC                | Blocked by ACL                        | Passed |
| Guest PC to Internet             | Successful                            | Passed |
| HR PC to Internet                | Successful NAT translation            | Passed |
| HSRP failover                    | Standby switch becomes active gateway | Passed |
| EtherChannel member link failure | Connectivity remains available        | Passed |
| STP redundant path recovery      | Alternate path becomes forwarding     | Passed |

---

## Failover Validation

### HSRP Failover

One active SVI was shut down to verify that the standby multilayer switch takes over the virtual gateway role.

### EtherChannel Failover

One EtherChannel member link was shut down to confirm that traffic continues through the remaining active link.

### STP Failover

One trunk uplink was shut down to validate STP path recalculation and redundant path recovery.

---

## Project Outcome

The project successfully demonstrates a secure, redundant, and scalable enterprise campus network. It includes core CCNA-level technologies such as VLANs, trunking, EtherChannel, STP, inter-VLAN routing, HSRP, OSPF, DHCP, NAT, ACLs, port security, and failover validation.

This project helped strengthen practical understanding of enterprise LAN design, routing, switching, network security, and high availability concepts.

---

## Author

Aman Mallik

```
```

