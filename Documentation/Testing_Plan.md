# Testing Plan

## Objective

The purpose of this testing plan is to verify VLAN segmentation, trunking, EtherChannel, STP, HSRP, OSPF, DHCP, NAT, ACL security, port security, and failover functionality in the enterprise campus network.

---

## Verification Commands

| Feature | Device | Command |
|---|---|---|
| VLANs | ACC1, ACC2, DSW1, DSW2 | `show vlan brief` |
| Trunks | ACC1, ACC2, DSW1, DSW2 | `show interfaces trunk` |
| EtherChannel | DSW1, DSW2 | `show etherchannel summary` |
| STP/PVST | ACC1, ACC2, DSW1, DSW2 | `show spanning-tree` |
| HSRP | DSW1, DSW2 | `show standby brief` |
| OSPF | R1, DSW1, DSW2 | `show ip ospf neighbor` |
| Routing | R1, DSW1, DSW2 | `show ip route` |
| DHCP | DSW1 | `show ip dhcp binding` |
| NAT | R1 | `show ip nat translations` |
| ACL | DSW1, DSW2 | `show access-lists` |
| Port Security | ACC1, ACC2 | `show port-security` |

---

## Connectivity Test Cases

| Test Case | Source | Destination | Expected Result |
|---|---|---|---|
| HR to Finance | VLAN 10 PC | VLAN 20 PC | Success |
| HR to IT | VLAN 10 PC | VLAN 30 PC | Success |
| Finance to Server | VLAN 20 PC | VLAN 100 Server | Success |
| Guest to HR | VLAN 40 PC | VLAN 10 PC/Gateway | Fail |
| Guest to Finance | VLAN 40 PC | VLAN 20 PC/Gateway | Fail |
| Guest to Server | VLAN 40 PC | VLAN 100 Server/Gateway | Fail |
| Guest to Internet | VLAN 40 PC | 8.8.8.8 | Success |
| HR to Internet | VLAN 10 PC | 8.8.8.8 | Success |

---

## ACL Testing

The Guest VLAN is restricted using an extended ACL applied inbound on VLAN 40 SVI.

### Commands tested from Guest PC

```bash
ping 192.168.10.65
ping 192.168.10.129
ping 192.168.10.145
ping 8.8.8.8
