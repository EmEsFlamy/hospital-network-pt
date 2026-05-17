# Network Design Documentation
## Hospital Network Simulation — Example Hospital Environment

---

## 1. Project Overview

This document describes the design decisions, addressing scheme, and configuration rationale for a hospital network simulation built in Cisco Packet Tracer. The project was completed as part of CompTIA Network+ preparation and reflects real-world hospital network segmentation principles.

---

## 2. Network Topology

```
                    [CORE-ROUTER]
                    Gig0/0 (trunk)
                         |
                    [CORE-SW 3560]
                    /      |      \
              Fa0/1     Fa0/2     Fa0/3
                 |         |         |
          [SW-MEDICAL] [SW-ADMIN] [SW-GUEST]
           /       \    /      \   /    |   \
        [PC0]    [PC1] [PC2] [PC3][PC4][PC5][AP-GUEST]

        CORE-SW Fa0/24 → [SRV-DHCP]
```

---

## 3. VLAN Design

| VLAN ID | Name | Purpose |
|---|---|---|
| 10 | MEDICAL | Medical staff workstations |
| 20 | ADMIN | Administrative staff workstations |
| 30 | GUEST | Patient and visitor Wi-Fi (via AP-GUEST) |
| 99 | MANAGEMENT | Server infrastructure, network management |

**Design rationale:** Each department is isolated in its own VLAN to limit lateral movement in case of a security incident. Proper segmentation limits blast radius — if one VLAN is compromised, the attacker cannot freely move to other network segments.

---

## 4. IP Addressing Scheme

| VLAN | Network | Subnet Mask | Gateway | DHCP Range |
|---|---|---|---|---|
| 10 MEDICAL | 192.168.10.0/24 | 255.255.255.0 | 192.168.10.1 | 192.168.10.100–254 |
| 20 ADMIN | 192.168.20.0/24 | 255.255.255.0 | 192.168.20.1 | 192.168.20.100–254 |
| 30 GUEST | 192.168.30.0/24 | 255.255.255.0 | 192.168.30.1 | 192.168.30.100–254 |
| 99 MANAGEMENT | 192.168.99.0/24 | 255.255.255.0 | 192.168.99.1 | Static only |

### Device Addresses

| Device | Interface | IP Address |
|---|---|---|
| CORE-ROUTER | Gig0/0.10 | 192.168.10.1 |
| CORE-ROUTER | Gig0/0.20 | 192.168.20.1 |
| CORE-ROUTER | Gig0/0.30 | 192.168.30.1 |
| CORE-ROUTER | Gig0/0.99 | 192.168.99.1 |
| SRV-DHCP | Fa0/24 | 192.168.99.2 |

---

## 5. Inter-VLAN Routing — Router-on-a-Stick

**Why Router-on-a-Stick?**
A single physical GigabitEthernet port (Gig0/0) on the router handles all inter-VLAN traffic through logical subinterfaces. This is cost-effective and appropriate for a small-to-medium hospital environment where a Layer 3 switch is not available.

**Trunk configuration:**
- CORE-SW Gig0/1 → CORE-ROUTER Gig0/0: dot1q trunk
- CORE-SW Fa0/1, Fa0/2, Fa0/3 → access switches: dot1q trunk

---

## 6. Access Control Lists

**ACL Name:** `BLOCK-GUEST`  
**Type:** Extended  
**Applied:** Inbound on Gig0/0.30

```
deny ip 192.168.30.0 0.0.0.255 192.168.10.0 0.0.0.255
deny ip 192.168.30.0 0.0.0.255 192.168.20.0 0.0.0.255
permit ip any any
```

**Why Extended ACL inbound on Gig0/0.30?**
Extended ACLs should be placed as close to the source as possible. Applying inbound on the GUEST subinterface ensures packets are dropped before the router processes them — saving resources and blocking traffic at the earliest point.

**Why not block MEDICAL ↔ ADMIN?**
Internal hospital departments need to communicate (shared systems, printers, file servers). Only the untrusted guest network requires full isolation.

---

## 7. DHCP Configuration

**Server:** SRV-DHCP (192.168.99.2)  
**Relay:** `ip helper-address 192.168.99.2` configured on Gig0/0.10, Gig0/0.20, Gig0/0.30

**Why DHCP Relay?**
DHCP uses broadcast — broadcasts do not cross router boundaries. The `ip helper-address` command converts DHCP broadcasts to unicast packets directed at the server, allowing a single DHCP server to serve multiple VLANs.

---

## 8. Port Security

**Applied on:** SW-MEDICAL (Fa0/2, Fa0/3), SW-ADMIN (Fa0/2, Fa0/3)  
**Not applied on:** SW-GUEST

| Setting | Value | Rationale |
|---|---|---|
| Maximum MAC | 1 | One device per port |
| Violation mode | Shutdown | Unauthorized device = port disabled immediately |
| MAC learning | Sticky | Switch learns and saves authorized MAC automatically |

**Why Shutdown and not Restrict?**
In a medical environment, an unauthorized device is a potential attack vector. Shutdown is the most aggressive response — it forces manual intervention by IT before the port is re-enabled, creating an audit trail.

**Why not on SW-GUEST?**
Guest network is dynamic — different devices connect at different times. Port Security with sticky MAC and maximum 1 would block legitimate users after the first device. Guest traffic isolation is handled by ACL instead.

---

## 9. NTP & Syslog

**NTP Server:** SRV-DHCP (192.168.99.2), Stratum 1  
**Syslog Server:** SRV-DHCP (192.168.99.2)  
**Configured on:** CORE-ROUTER, CORE-SW, SW-MEDICAL, SW-ADMIN, SW-GUEST

**Why NTP?**
Without synchronized clocks, log correlation across devices is unreliable. If a security incident occurs, timestamps from different devices must match to reconstruct the attack timeline — a fundamental requirement for SOC operations.

**Why centralized Syslog?**
Device logs are lost on reboot. A central Syslog server retains all events permanently. In a real hospital environment, centralized logs would capture unusual account activity before a security incident escalates — a key advantage for early threat detection.

---

## 10. Troubleshooting Log

| Issue | Cause | Resolution |
|---|---|---|
| Trunk command rejected on 3560 | Encapsulation set to Auto by default | Added `switchport trunk encapsulation dot1q` before `switchport mode trunk` |
| Red triangles between router and core switch | Router interfaces shutdown by default | Applied `no shutdown` on Gig0/0 |
| DHCP request failed on PC | DHCP service was Off on server | Enabled DHCP service on SRV-DHCP |
| Sticky MAC Addresses showing 0 | No traffic generated after Port Security config | Pinged gateway from each PC to trigger MAC learning |
| `logging trap informational` not recognized | Command not supported in Packet Tracer | Removed command, used `logging [ip]` only |

---

## 11. Lessons Learned

- **Encapsulation matters:** 3560 requires explicit dot1q encapsulation before trunk mode — unlike 2960 which handles it automatically.
- **Implicit deny:** ACL always ends with an implicit `deny all`. Without `permit ip any any`, all traffic would be blocked — not just GUEST.
- **DHCP Relay is essential:** A single DHCP server cannot serve multiple VLANs without `ip helper-address` — broadcasts do not cross Layer 3 boundaries.
- **NTP before Syslog:** Time synchronization must be configured before Syslog is useful — otherwise log timestamps are meaningless for incident correlation.
- **Design decisions have security implications:** Choosing Shutdown over Restrict for Port Security is a conscious security decision, not just a configuration choice.
