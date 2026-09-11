# Enterprise VLAN + DHCP Relay Lab (Cisco Packet Tracer / PNET)

A small enterprise-style network lab built to practice **VLAN segmentation, trunk security, inter-VLAN routing via SVIs, and DHCP relay to a Windows Server** across VLANs.

![Topology](topology.png)

## 🧱 Topology Overview

| Device | Role | Notes |
|---|---|---|
| **Switch (Core)** | Multilayer Switch / L3 core | Inter-VLAN routing (SVIs), trunk aggregation |
| **SW** | Access Switch | Trunk to Core, access ports for VLAN 10/20/30 |
| **SW2** | Access Switch | Trunk to Core, access ports (VLAN 10/20/30 group) |
| **sw3** | Access Switch | Connects Windows Server (VLAN 100) |
| **VPC5–VPC10** | End hosts | DHCP clients across VLAN 10/20/30 |
| **Winserver** | DHCP Server | Lives in VLAN 100 (dedicated services VLAN) |

## 🎯 Lab Objectives

- Segment the network into **VLAN 10, VLAN 20, VLAN 30** (user VLANs) and **VLAN 100** (services/DHCP VLAN)
- Configure **manual (static) 802.1Q trunks** between Core ↔ Access switches
- Centralize **DHCP on Windows Server**, relayed across VLANs using `ip helper-address`
- Use the Core multilayer switch as the **default gateway** for every VLAN via SVIs
- **Harden trunk ports** against DTP-based VLAN hopping / rogue trunk attacks

## 🔐 Security Hardening

The main risk in a manually-trunked topology is an attacker (or misconfigured device) triggering **Dynamic Trunking Protocol (DTP)** negotiation on an access port to turn it into a trunk and gain access to every VLAN on the segment. This lab locks that down:

- `switchport nonegotiate` on **every** port (trunk and access) — DTP negotiation is fully disabled network-wide, so no port can be talked into becoming a trunk.
- `switchport mode trunk` set explicitly (never `dynamic desirable/auto`) on inter-switch links.
- `switchport trunk allowed vlan` pruned to only the VLANs actually needed on each link.
- Access ports locked with **port-security** (sticky MAC, max 1) to prevent MAC spoofing/rogue devices.
- **BPDU Guard** + **PortFast** on all access ports to block rogue switches joining the STP domain.
- Unused physical ports shut down and parked in a black-hole VLAN.

See [`configs/`](./configs) for full running-config snippets (Core, SW, SW2, sw3).

## 🌐 Addressing Plan

| VLAN | Purpose | Network | Gateway (SVI) |
|---|---|---|---|
| 10 | Users A | 192.168.10.0/24 | 192.168.10.1 |
| 20 | Users B | 192.168.20.0/24 | 192.168.20.1 |
| 30 | Users C | 192.168.30.0/24 | 192.168.30.1 |
| 100 | Services / DHCP Server | 192.168.100.0/24 | 192.168.100.1 |

DHCP requests from VLAN 10/20/30 clients are relayed via `ip helper-address 192.168.100.10` configured on each respective SVI, forwarding broadcast DHCP DISCOVER traffic as unicast to the Windows DHCP Server in VLAN 100.

## ✅ Verification Checklist

- [ ] `show interfaces trunk` confirms only intended trunks are up, allowed VLANs pruned correctly
- [ ] `show port-security interface <intf>` shows learned/secured MAC per access port
- [ ] VPC5/6/7/8/9/10 successfully receive DHCP leases from the correct VLAN scope
- [ ] Inter-VLAN ping works through Core SVIs
- [ ] Attempting to force trunk negotiation on an access port fails (port stays access-mode)

## 🛠️ Tools Used

- Cisco Packet Tracer (PNET labeled topology)
- Windows Server (DHCP role, multi-scope)
- Cisco IOS (Multilayer Switch + Layer 2 Access Switches)

---

*Part of my ongoing network engineering practice labs — feedback and suggestions welcome.*
