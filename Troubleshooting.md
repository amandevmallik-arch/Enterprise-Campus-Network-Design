````markdown
# Troubleshooting Notes

## Project: Enterprise Campus Network Design

This document contains common troubleshooting scenarios encountered during the enterprise campus network implementation. The project includes VLANs, trunking, EtherChannel, STP/PVST, HSRP, OSPF, DHCP, NAT, ACLs, port security, and failover testing.

---

## Issue 1: PC Did Not Receive DHCP Address

### Symptoms

- PC received an APIPA address such as `169.254.x.x`.
- PC did not receive an IP address from the correct VLAN subnet.
- PC could not ping its default gateway.

### Verification Commands

Run on the access switch:

```bash
show vlan brief
````

Run on the DHCP server device, usually DSW1:

```bash
show ip dhcp binding
show ip dhcp pool
```

### Possible Causes

* PC port assigned to the wrong VLAN.
* VLAN not created on the switch.
* Trunk link not allowing the required VLAN.
* DHCP pool configured with the wrong subnet or default gateway.
* SVI for that VLAN is down.

### Resolution

* Verified the PC-facing port VLAN assignment.
* Confirmed that the VLAN exists on the access and distribution switches.
* Checked trunk allowed VLANs.
* Corrected the DHCP pool network and default-router values.
* Renewed DHCP on the PC.

Example fix:

```bash
interface fa0/7
 switchport mode access
 switchport access vlan 40
 no shutdown
```

---

## Issue 2: VLAN Was Missing from Trunk Link

### Symptoms

* PCs in a VLAN could not reach their default gateway.
* VLAN existed on the switch, but traffic was not passing between access and distribution switches.
* `show interfaces trunk` did not show the required VLAN in the allowed VLAN list.

### Verification Command

Run on access and distribution switches:

```bash
show interfaces trunk
```

### Possible Causes

* Required VLAN not allowed on trunk.
* Trunk was not configured on both sides.
* Interface was still in access mode.

### Resolution

Configured trunk mode and allowed the required VLANs.

Example:

```bash
interface fa0/1
 switchport mode trunk
 switchport trunk allowed vlan 10,20,30,40,99,100
 no shutdown
```

---

## Issue 3: EtherChannel Did Not Form Properly

### Symptoms

* `show etherchannel summary` did not show `Po10(SU)`.
* One or more member ports were in suspended or standalone state.
* Traffic was not passing correctly between DSW1 and DSW2.

### Verification Command

Run on DSW1 and DSW2:

```bash
show etherchannel summary
```

### Possible Causes

* Mismatched trunk settings on member interfaces.
* Different allowed VLAN lists on each side.
* Incorrect channel-group mode.
* One side configured as LACP active while the other side was not compatible.
* Physical interfaces were not identical in speed/duplex/trunk mode.

### Resolution

Configured identical settings on both physical member interfaces and the Port-channel interface.

Example:

```bash
interface range fa0/23 - 24
 switchport mode trunk
 switchport trunk allowed vlan 10,20,30,40,99,100
 channel-group 10 mode active
 no shutdown

interface port-channel 10
 switchport mode trunk
 switchport trunk allowed vlan 10,20,30,40,99,100
 no shutdown
```

Expected output:

```text
Po10(SU)
```

---

## Issue 4: STP Blocking Unexpected Link

### Symptoms

* One access switch uplink was blocking.
* Some VLAN traffic preferred an unexpected path.
* Redundant links were not forwarding as expected.

### Verification Commands

Run on DSW1, DSW2, ACC1, and ACC2:

```bash
show spanning-tree
show spanning-tree vlan 10
show spanning-tree vlan 20
```

### Possible Causes

* STP root bridge priority not configured.
* Wrong switch became root bridge for a VLAN.
* PVST load balancing was not aligned with HSRP active gateway.

### Resolution

Configured DSW1 and DSW2 as root bridges for different VLAN groups.

Example:

On DSW1:

```bash
spanning-tree vlan 10,30,99,100 root primary
spanning-tree vlan 20,40 root secondary
```

On DSW2:

```bash
spanning-tree vlan 20,40 root primary
spanning-tree vlan 10,30,99,100 root secondary
```

---

## Issue 5: HSRP Gateway Was Not Active/Standby Correctly

### Symptoms

* `show standby brief` did not show expected active and standby devices.
* PCs could not ping the virtual gateway.
* Both distribution switches showed unexpected HSRP state.

### Verification Command

Run on DSW1 and DSW2:

```bash
show standby brief
```

### Possible Causes

* HSRP virtual IP not configured correctly.
* Wrong priority configured.
* `standby preempt` missing.
* VLAN SVI down.
* VLAN not active on trunk or no active port in that VLAN.

### Resolution

Verified SVI IP addresses, HSRP virtual IPs, priorities, and preempt configuration.

Example:

```bash
interface vlan 40
 ip address 192.168.10.3 255.255.255.192
 standby 40 ip 192.168.10.1
 standby 40 priority 110
 standby 40 preempt
 no shutdown
```

---

## Issue 6: OSPF Neighbor Was Not Forming

### Symptoms

* `show ip ospf neighbor` showed no neighbors.
* DSW1/DSW2 did not learn routes from R1.
* R1 did not learn VLAN routes from distribution switches.

### Verification Commands

Run on R1, DSW1, and DSW2:

```bash
show ip ospf neighbor
show ip route ospf
show ip interface brief
```

### Possible Causes

* Routed interfaces were down.
* Wrong IP address or subnet mask on routed links.
* OSPF network statement was missing or incorrect.
* Router IDs were not configured.
* Interfaces were not in the same OSPF area.

### Resolution

Corrected routed link IPs and OSPF area 0 network statements.

Example:

```bash
router ospf 10
 router-id 1.1.1.1
 network 10.255.1.0 0.0.0.3 area 0
 network 10.255.1.4 0.0.0.3 area 0
```

---

## Issue 7: NAT Was Not Working

### Symptoms

* Internal VLAN PCs could not ping the ISP loopback `8.8.8.8`.
* `show ip nat translations` showed no translations.
* R1 could ping ISP, but PCs could not reach internet.

### Verification Commands

Run on R1:

```bash
show ip route
show ip nat translations
show ip nat statistics
show access-lists
```

### Possible Causes

* NAT inside/outside interfaces incorrectly configured.
* Default route missing on R1.
* NAT ACL did not match internal subnets.
* OSPF default route not advertised to DSW1/DSW2.
* Return route missing on ISP.

### Resolution

Configured NAT inside on LAN-facing interfaces and NAT outside on ISP-facing interface.

Example:

```bash
interface g0/0
 ip nat inside

interface g0/1
 ip nat inside

interface serial0/0/0
 ip nat outside

access-list 1 permit 192.168.10.0 0.0.0.255
ip nat inside source list 1 interface serial0/0/0 overload

ip route 0.0.0.0 0.0.0.0 203.0.113.1
```

---

## Issue 8: Guest VLAN Could Access Internal VLANs

### Symptoms

* Guest PC was able to ping HR, Finance, IT, Management, or Server VLANs.
* ACL existed but did not block traffic.
* `show access-lists` showed no hit count increase.

### Verification Commands

Run on DSW1 and DSW2:

```bash
show access-lists
show running-config interface vlan 40
```

### Possible Causes

* ACL was created but not applied to `interface vlan 40`.
* ACL applied in the wrong direction.
* ACL applied only on one HSRP switch.
* Guest PC was not actually in VLAN 40.
* Guest PC had an incorrect IP address or default gateway.

### Resolution

Applied the ACL inbound on VLAN 40 SVI on both DSW1 and DSW2.

Example:

```bash
ip access-list extended GUEST_RESTRICT
 deny ip 192.168.10.0 0.0.0.63 192.168.10.64 0.0.0.31
 deny ip 192.168.10.0 0.0.0.63 192.168.10.96 0.0.0.31
 deny ip 192.168.10.0 0.0.0.63 192.168.10.128 0.0.0.15
 deny ip 192.168.10.0 0.0.0.63 192.168.10.144 0.0.0.15
 deny ip 192.168.10.0 0.0.0.63 192.168.10.160 0.0.0.15
 permit ip 192.168.10.0 0.0.0.63 any

interface vlan 40
 ip access-group GUEST_RESTRICT in
```

Expected test result from Guest PC:

```text
ping 192.168.10.65   = fail
ping 192.168.10.129  = fail
ping 192.168.10.145  = fail
ping 8.8.8.8         = success
```

---

## Issue 9: Port Security Violation

### Symptoms

* A PC connected to an access port could not communicate.
* `show port-security` showed violation count increasing.
* Port security blocked traffic from a new MAC address.

### Verification Commands

Run on ACC1 or ACC2:

```bash
show port-security
show port-security interface fa0/3
```

### Possible Causes

* A different PC was connected to a port where sticky MAC was already learned.
* Maximum MAC address limit was set to 1.
* Sticky MAC address was learned from a previous device.
* Port security was applied on a trunk port by mistake.

### Resolution

Verified the connected PC and cleared or relearned the sticky MAC address if needed.

Example configuration:

```bash
interface fa0/3
 switchport mode access
 switchport port-security
 switchport port-security maximum 1
 switchport port-security mac-address sticky
 switchport port-security violation restrict
```

---

## Issue 10: Failover Test Did Not Work

### Symptoms

* HSRP standby switch did not become active.
* EtherChannel went down completely after one member link failed.
* STP alternate path did not recover.

### Verification Commands

```bash
show standby brief
show etherchannel summary
show spanning-tree
show interfaces trunk
```

### Possible Causes

* HSRP not configured on both distribution switches.
* EtherChannel member ports had mismatched settings.
* STP root bridge and redundant uplinks were not configured properly.
* VLAN was not allowed on backup trunk path.

### Resolution

Verified HSRP, EtherChannel, and STP configuration on both distribution and access layer switches.

Failover tests performed:

```text
HSRP Failover       : Shut down active VLAN SVI and verified standby takeover.
EtherChannel Test   : Shut down one member link and verified Port-channel stayed up.
STP Failover        : Shut down one uplink and verified alternate path recovery.
```

---

## Final Troubleshooting Summary

| Feature       | Main Command Used           | Status   |
| ------------- | --------------------------- | -------- |
| VLANs         | `show vlan brief`           | Verified |
| Trunks        | `show interfaces trunk`     | Verified |
| EtherChannel  | `show etherchannel summary` | Verified |
| STP/PVST      | `show spanning-tree`        | Verified |
| HSRP          | `show standby brief`        | Verified |
| OSPF          | `show ip ospf neighbor`     | Verified |
| DHCP          | `show ip dhcp binding`      | Verified |
| NAT           | `show ip nat translations`  | Verified |
| ACL           | `show access-lists`         | Verified |
| Port Security | `show port-security`        | Verified |

---

## Conclusion

The enterprise campus network was successfully troubleshot and validated across Layer 2, Layer 3, security, routing, NAT, and redundancy features. The final configuration supports secure VLAN segmentation, inter-VLAN routing, redundant default gateways, dynamic routing, controlled guest access, internet connectivity, and high availability through failover mechanisms.

```
```
