# 🌐 Lab 6.8.1 — Configure NAT for IPv4

## 📌 Lab Info

| Field | Details |
|-------|---------|
| **Course** | CCNA — Network Address Translation |
| **Lab** | 6.8.1 |
| **Tool** | Cisco Packet Tracer |
| **Difficulty** | ⭐⭐⭐⭐☆ Advanced |
| **Score** | ✅ Complete |
| **Topic** | Dynamic NAT with PAT + Static NAT — Combined Lab |

> 📥 **Download the Packet Tracer file** and test the configuration yourself.

---

## 🎯 Objectives

- Configure **Dynamic NAT with PAT** for three internal LANs
- Configure **Static NAT** for an internal server
- Use a **named ACL** to define permitted traffic
- Configure NAT interfaces correctly

---

## 🗺️ Topology Overview

> 📷 See `topology.png` for the full network diagram.

The topology includes:
- **R2** — the only router being configured in this lab
- **LAN1, LAN2, LAN3** — three internal networks using PAT
- **local.pka server** — internal server mapped with Static NAT
- **Outside network** — `209.165.202.128/30`

---

## 🔑 Address Space Planning

From `209.165.202.128/30`:

| Address | Role |
|---------|------|
| `209.165.202.128` | Network address |
| `209.165.202.129` | **R2POOL** — PAT pool address |
| `209.165.202.130` | **Static NAT** — server public address |
| `209.165.202.131` | Broadcast |

---

## 📋 Important Lab Rules

> ⚠️ Names must match **exactly** — case-sensitive!

| Item | Required Name |
|------|--------------|
| Named ACL | `R2NAT` |
| NAT Pool | `R2POOL` |
| LAN order in ACL | LAN1 → LAN2 → LAN3 |

---

## ⚙️ Complete Configuration — R2 Only

### Step 1 — Named Standard ACL (R2NAT)

Permit LAN1, LAN2, and LAN3 **in this exact order**:

```
R2(config)# ip access-list standard R2NAT
R2(config-std-nacl)# permit [LAN1-network] [wildcard]
R2(config-std-nacl)# permit [LAN2-network] [wildcard]
R2(config-std-nacl)# permit [LAN3-network] [wildcard]
R2(config-std-nacl)# exit
```

---

### Step 2 — NAT Pool (R2POOL) — First address only

```
R2(config)# ip nat pool R2POOL 209.165.202.129 209.165.202.129 netmask 255.255.255.252
```

> 💡 Both start and end IP are `209.165.202.129` because the pool has only **one address**.
> PAT allows all three LANs to share this single IP using port numbers.

---

### Step 3 — Bind ACL to Pool with Overload (PAT)

```
R2(config)# ip nat inside source list R2NAT pool R2POOL overload
```

---

### Step 4 — Static NAT for the Server — Second address

```
R2(config)# ip nat inside source static [server-inside-IP] 209.165.202.130
```

---

### Step 5 — Configure NAT Interfaces

```
R2(config)# interface [inside-interface]
R2(config-if)# ip nat inside
R2(config-if)# exit

R2(config)# interface [outside-interface]
R2(config-if)# ip nat outside
R2(config-if)# exit
```

---

## 🔄 How This Lab Combines Both NAT Types

```
[LAN1 hosts]  ─────────────────────────────────→  PAT → 209.165.202.129
[LAN2 hosts]  ──────→  R2NAT ACL → R2POOL     →  PAT → 209.165.202.129
[LAN3 hosts]  ─────────────────────────────────→  PAT → 209.165.202.129

[local.pka]   ──────→  Static NAT             →  Fixed → 209.165.202.130
```

---

## ✅ Verification Commands

```bash
# View all NAT translations
show ip nat translations

# Check NAT statistics
show ip nat statistics

# Verify ACL R2NAT
show access-lists R2NAT

# Verify interfaces
show ip interface brief
```

### Expected output — `show ip nat translations`

```
Pro  Inside global          Inside local         Outside local        Outside global
---  209.165.202.130        [server-local-IP]     ---                  ---
tcp  209.165.202.129:1025   [LAN1-host]:1025      X.X.X.X:80           X.X.X.X:80
tcp  209.165.202.129:1026   [LAN2-host]:1026      X.X.X.X:80           X.X.X.X:80
tcp  209.165.202.129:1027   [LAN3-host]:1027      X.X.X.X:80           X.X.X.X:80
```

> The **static entry** is always present.
> The **PAT entries** appear only during active sessions.

---

## 💡 Key Concepts Summary

| Concept | Details |
|---------|---------|
| **Named ACL** | `ip access-list standard R2NAT` — must match exactly |
| **NAT Pool** | `R2POOL` — single IP `209.165.202.129` |
| **PAT (overload)** | All 3 LANs share one public IP via port numbers |
| **Static NAT** | Server always mapped to `209.165.202.130` — fixed |
| **Order in ACL** | LAN1 → LAN2 → LAN3 — must follow this exact order for scoring |
| **First address** | `209.165.202.129` → PAT pool |
| **Second address** | `209.165.202.130` → Static NAT server |

---

## 📊 Dynamic PAT vs Static NAT — In This Lab

| | Dynamic PAT (R2POOL) | Static NAT (Server) |
|--|---------------------|---------------------|
| **Public IP** | `209.165.202.129` | `209.165.202.130` |
| **Inside hosts** | LAN1 + LAN2 + LAN3 | local.pka server only |
| **Mapping** | Dynamic — port-based | Fixed — permanent |
| **Direction** | Inside → Outside | Both directions |
| **Purpose** | Internet access for clients | Server reachable from outside |

---

## 📝 What I Learned

- How to combine **PAT and Static NAT** on the same router efficiently
- Why ACL and pool names must match exactly — Packet Tracer is case-sensitive
- How a single IP address in a pool still allows thousands of simultaneous sessions via PAT
- Why the server needs **Static NAT** — it must always have the same public IP so outside hosts can reach it consistently
- The importance of ACL statement order — Packet Tracer checks them in the specified sequence
- How the `/30` subnet gives exactly 2 usable IPs — perfectly split between PAT and Static NAT
