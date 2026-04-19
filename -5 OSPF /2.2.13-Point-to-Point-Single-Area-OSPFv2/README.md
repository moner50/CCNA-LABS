# 🔁 Lab 2.2.13 — Point-to-Point Single Area OSPFv2 Configuration

## 📌 Lab Info

| Field | Details |
|-------|---------|
| **Course** | CCNA — Routing & Switching |
| **Module** | 2.2.13 |
| **Tool** | Cisco Packet Tracer |
| **Difficulty** | ⭐⭐☆☆☆ Beginner–Intermediate |
| **Topic** | Single-Area OSPFv2 on Point-to-Point Links |

---

## 🎯 Objectives

1. Configure OSPFv2 on all routers in a single area (Area 0)
2. Assign router IDs manually
3. Advertise all connected networks using the `network` command
4. Verify full OSPF neighbor adjacency between routers
5. Confirm OSPF routes appear in the routing table

---

## 🗺️ Topology Overview

> 📷 See `topology.png` for the full diagram.

The topology consists of multiple routers connected via **point-to-point serial or Ethernet links**, all participating in **OSPF Area 0** (the backbone area).
Point-to-point links do not require a DR/BDR election, which simplifies the adjacency process.

---

## ⚙️ Configuration Summary

### Step 1 — Enable OSPF and assign a Router ID

```
Router(config)# router ospf 1
Router(config-router)# router-id X.X.X.X
```

> The Router ID uniquely identifies each router in OSPF.
> Best practice: use a loopback or manually assign it (e.g., `1.1.1.1`, `2.2.2.2`).

---

### Step 2 — Advertise Networks

```
Router(config-router)# network [network-address] [wildcard-mask] area 0
```

> Use wildcard masks (inverse of subnet mask).
> Example: for `192.168.1.0/24` → wildcard is `0.0.0.255`

---

### Step 3 — Passive Interface (optional but recommended)

```
Router(config-router)# passive-interface [interface]
```

> Apply on interfaces connected to end devices (no OSPF neighbors).
> Prevents unnecessary Hello packets from being sent.

---

## ✅ Verification Commands

```bash
# Check OSPF neighbor relationships
show ip ospf neighbor

# Verify OSPF routes in routing table (marked with O)
show ip route ospf

# View OSPF process details
show ip ospf

# Check interface OSPF participation
show ip ospf interface brief
```

### Expected Output — `show ip ospf neighbor`

```
Neighbor ID     Pri   State           Dead Time   Address         Interface
2.2.2.2           0   FULL/  -        00:00:35    10.0.0.2        Serial0/0/0
3.3.3.3           0   FULL/  -        00:00:39    10.0.0.6        Serial0/0/1
```

> **State: FULL** means the adjacency is fully established ✅
> **Priority: 0** on point-to-point links — no DR/BDR election needed

---

## 💡 Key Concepts

| Concept | Explanation |
|---------|-------------|
| **OSPF Area 0** | The backbone area — all routers must connect to it |
| **Router ID** | A 32-bit value that uniquely identifies a router in OSPF |
| **Point-to-Point** | Link type with only 2 routers — no DR/BDR election |
| **Wildcard Mask** | Inverse of subnet mask, used in `network` statements |
| **FULL State** | Indicates a complete and working OSPF neighbor relationship |
| **OSPF Cost** | Metric based on bandwidth: Cost = 100 Mbps / Interface bandwidth |

---

## 📝 What I Learned

- How to enable and configure OSPFv2 from scratch on multiple routers
- The importance of the Router ID and how OSPF uses it
- Why point-to-point links skip the DR/BDR election process
- How to read and interpret `show ip ospf neighbor` output
- How OSPF routes appear in the routing table with the `O` code
