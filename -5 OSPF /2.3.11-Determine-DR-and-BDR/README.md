# 🏆 Lab 2.3.11 — Determine the DR and BDR

## 📌 Lab Info

| Field | Details |
|-------|---------|
| **Course** | CCNA — Routing & Switching |
| **Module** | 2.3.11 |
| **Tool** | Cisco Packet Tracer |
| **Difficulty** | ⭐⭐⭐☆☆ Intermediate |
| **Topic** | OSPF DR/BDR Election on Multi-Access Networks |

---

## 🎯 Objectives

1. Understand why DR and BDR are elected on multi-access networks
2. Observe the default OSPF DR/BDR election process
3. Influence the election using OSPF priority and Router ID
4. Verify DR/BDR roles using show commands

---

## 🗺️ Topology Overview

> 📷 See `topology.png` for the full diagram.

The topology uses a **multi-access network** (such as Ethernet/LAN) where multiple routers connect to the same segment. On these networks, OSPF elects a **Designated Router (DR)** and a **Backup Designated Router (BDR)** to reduce the number of OSPF adjacencies and LSA flooding.

---

## 🧠 Why DR/BDR?

On a multi-access network with **N routers**, without DR/BDR:
- Every router forms a full adjacency with every other router
- This creates **N(N-1)/2** adjacencies — inefficient!

With DR/BDR:
- All routers form adjacency **only with the DR and BDR**
- The DR redistributes LSAs to all others
- Much more scalable ✅

---

## 🗳️ DR/BDR Election Rules

| Priority | Rule |
|----------|------|
| **1st** | Highest OSPF interface priority (default: 1, range: 0–255) |
| **2nd** | Highest Router ID (if priority is tied) |
| **Priority = 0** | Router will NEVER become DR or BDR |

> ⚠️ **Important:** The election is NOT preemptive.
> If a new router with a higher priority joins later, it does NOT take over.
> You must reset OSPF (`clear ip ospf process`) to trigger a new election.

---

## ⚙️ Configuration Summary

### Set OSPF Priority to influence election

```
Router(config)# interface [interface]
Router(config-if)# ip ospf priority [0-255]
```

> Example: Set a router to always become DR
> ```
> ip ospf priority 255
> ```

> Example: Exclude a router from DR/BDR election
> ```
> ip ospf priority 0
> ```

---

### Reset OSPF process to trigger re-election (if needed)

```
Router# clear ip ospf process
```

---

## ✅ Verification Commands

```bash
# See DR and BDR for each interface
show ip ospf interface [interface]

# Check neighbor states and roles
show ip ospf neighbor

# View routing table for OSPF routes
show ip route ospf
```

### Expected Output — `show ip ospf neighbor`

```
Neighbor ID     Pri   State           Dead Time   Address         Interface
2.2.2.2           1   FULL/DR         00:00:36    192.168.1.2     GigE0/0
3.3.3.3           1   FULL/BDR        00:00:38    192.168.1.3     GigE0/0
4.4.4.4           1   FULL/DROTHER    00:00:31    192.168.1.4     GigE0/0
```

### Understanding the State Column

| State | Meaning |
|-------|---------|
| `FULL/DR` | This neighbor IS the Designated Router |
| `FULL/BDR` | This neighbor IS the Backup DR |
| `FULL/DROTHER` | This neighbor is neither DR nor BDR |
| `2WAY/DROTHER` | Two DROTHER routers — they do NOT form full adjacency with each other |

---

## 💡 Key Concepts

| Concept | Explanation |
|---------|-------------|
| **DR (Designated Router)** | Central point for LSA exchange on the segment |
| **BDR (Backup DR)** | Takes over if the DR fails |
| **DROTHER** | Any router that is neither DR nor BDR |
| **Multicast 224.0.0.6** | Used by DROTHERs to send updates to DR/BDR |
| **Multicast 224.0.0.5** | Used by DR to flood updates to all OSPF routers |
| **Non-preemptive** | A higher-priority router joining later won't replace the current DR |

---

## 📝 What I Learned

- Why OSPF needs a DR and BDR on multi-access (Ethernet) networks
- How the election works: priority first, then Router ID as tiebreaker
- How to manually control which router becomes DR using `ip ospf priority`
- The difference between FULL/DR, FULL/BDR, FULL/DROTHER, and 2WAY states
- Why DROTHERs only form FULL adjacency with DR and BDR, not with each other
