![Status](https://img.shields.io/badge/Status-In%20Progress-yellow)
![Tool](https://img.shields.io/badge/Tool-Cisco%20Packet%20Tracer-blue)
![Cert](https://img.shields.io/badge/Cert-CompTIA%20Network%2B-red)
![Stage](https://img.shields.io/badge/Stage-2%20of%204-lightgrey)
# Hospital Network Simulation 🏥

Simulation of a hospital network built in Cisco Packet Tracer as part of my CompTIA Network+ preparation.

## Project Goals
- Design a segmented hospital network using VLANs
- Apply subnetting to real-world infrastructure
- Configure inter-VLAN routing and basic ACLs
- Document the process for portfolio purposes

## Network Segments
| VLAN | Department |
|---|---|
| VLAN 10 | Medical Staff |
| VLAN 20 | Administration |
| VLAN 30 | Guest / Patient Wi-Fi |
| VLAN 99 | Management |

## Project Stages
- [x] Stage 1 – Physical topology
- [x] Stage 2 – VLANs & trunking
- [ ] Stage 3 – IP addressing & subnetting
- [ ] Stage 4 – Routing & ACLs

## Tools
- Cisco Packet Tracer 8.x
- CompTIA Network+ (Jason Dion, Udemy)

## Author
IT Systems Administrator | Studying Cybersecurity | Target: SOC Analyst

## Stage 1 – Physical Topology
![Physical Topology](screenshots/stage1-physical-topology.png)

## Stage 2 – VLAN Configuration
![Updated Physical Topology](screenshots/stage2-full-topology.png)

| Switch | VLAN | Name |
|---|---|---|
| SW-MEDICAL | 10 | MEDICAL |
| SW-ADMIN | 20 | ADMIN |
| SW-GUEST | 30 | GUEST |
| CORE-SW | 99 | MANAGEMENT |

![CORE-SW VLANs](screenshots/stage2-vlan-core-sw.png)
![SW-MEDICAL VLANs](screenshots/stage2-vlan-sw-medical.png)
![SW-ADMIN VLANs](screenshots/stage2-vlan-sw-admin.png)
![SW-GUEST VLANs](screenshots/stage2-vlan-sw-guest.png)
