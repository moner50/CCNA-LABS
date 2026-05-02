# 🌐 Lab 6.6.7 — Configure PAT (Port Address Translation)

## 📌 Lab Info

| Field | Details |
|-------|---------|
| **Course** | CCNA — Network Address Translation |
| **Lab** | 6.6.7 |
| **Tool** | Cisco Packet Tracer |
| **Difficulty** | ⭐⭐⭐☆☆ Intermediate |
| **Topic** | PAT — Dynamic NAT with Overload & Interface-Based PAT |

> 📥 **Download the Packet Tracer file** and test the configuration yourself.

---

## 🎯 Objectives

### Part 1 — Configure Dynamic NAT with Overload (R1)
- Permit inside traffic using ACL
- Define a NAT pool
- Bind ACL to pool with `overload` keyword
- Configure NAT interfaces

### Part 2 — Verify Dynamic NAT with Overload (R1)
- Test web access from all inside hosts
- View NAT translation table

### Part 3 — Configure PAT using an Interface (R2)
- Permit inside traffic using ACL
- Bind ACL directly to the outside interface with `overload`
- Configure NAT interfaces

### Part 4 — Verify PAT Interface Implementation (R2)
- Test web access from all inside hosts
- Compare NAT statistics between R1 and R2

---

## 🗺️ Topology Overview

> 📷 See `topology.png` for the full network diagram.

| Router | Inside Hosts | Inside Network | NAT Method |
|--------|-------------|----------------|------------|
| **R1** | PC1, L1, PC2, L2 | `172.16.0.0/16` | PAT with Pool |
| **R2** | PC3, L3, PC4, L4 | `172.17.0.0/16` | PAT with Interface |

---

## 🧠 NAT vs Dynamic NAT vs PAT

| Feature | Static NAT | Dynamic NAT | PAT (Overload) |
|---------|-----------|-------------|----------------|
| Mapping | One-to-one fixed | One-to-one from pool | Many-to-one |
| Pool needed | ❌ | ✅ | Optional |
| Simultaneous users | 1 per IP | Limited by pool size | **Thousands** |
| Uses port numbers | ❌ | ❌ | ✅ |
| Keyword | — | — | **overload** |

> 💡 PAT is why your home router can have **one public IP** and still connect all your devices to the internet simultaneously!

---

## ⚙️ Configuration

---

### Part 1 — R1: Dynamic NAT with Overload (PAT Pool)

#### Step 1 — ACL to permit inside traffic
```
R1(config)# access-list 1 permit 172.16.0.0 0.0.255.255
```

#### Step 2 — Define the NAT pool
```
R1(config)# ip nat pool ANY_POOL_NAME 209.165.200.233 209.165.200.234 netmask 255.255.255.252
```

> The `/30` gives us 2 usable IPs: `209.165.200.233` and `209.165.200.234`

#### Step 3 — Bind ACL to pool with overload ← KEY DIFFERENCE from Dynamic NAT
```
R1(config)# ip nat inside source list 1 pool ANY_POOL_NAME overload
```

> 💡 The **`overload`** keyword is what makes this PAT instead of Dynamic NAT.
> It allows **multiple inside hosts to share the same public IP** by using different port numbers.

#### Step 4 — Configure NAT interfaces
```
R1(config)# interface s0/1/0
R1(config-if)# ip nat outside

R1(config)# interface g0/0/0
R1(config-if)# ip nat inside

R1(config)# interface g0/0/1
R1(config-if)# ip nat inside
```

---

### Part 3 — R2: PAT using an Interface (No Pool Needed)

#### Step 1 — ACL to permit inside traffic
```
R2(config)# access-list 2 permit 172.17.0.0 0.0.255.255
```

#### Step 2 — Bind ACL directly to outside interface with overload
```
R2(config)# ip nat inside source list 2 interface s0/1/1 overload
```

> 💡 Instead of a pool, R2 uses its **own outside interface IP** as the single public address.
> All inside hosts share that one IP — differentiated only by port numbers.

#### Step 3 — Configure NAT interfaces
```
R2(config)# interface s0/1/1
R2(config-if)# ip nat outside

R2(config)# interface g0/0/0
R2(config-if)# ip nat inside

R2(config)# interface g0/0/1
R2(config-if)# ip nat inside
```

---

## ✅ Verification

### View NAT translations
```
R1# show ip nat translations
R2# show ip nat translations
```

**Expected output on R1 (all 4 hosts using just ONE pool address):**
```
Pro  Inside global          Inside local         Outside local        Outside global
tcp  209.165.200.233:1025   172.16.1.2:1025      209.165.200.226:80   209.165.200.226:80
tcp  209.165.200.233:1026   172.16.1.3:1026      209.165.200.226:80   209.165.200.226:80
tcp  209.165.200.233:1027   172.16.2.2:1027      209.165.200.226:80   209.165.200.226:80
tcp  209.165.200.233:1028   172.16.2.3:1028      209.165.200.226:80   209.165.200.226:80
```

> Notice all 4 hosts share **`209.165.200.233`** — only the **port number** differs! ✅

---

### Compare NAT statistics
```
R1# show ip nat statistics
R2# show ip nat statistics
```

---

## ❓ Lab Questions & Answers

**Q1: Were all connections from PC1, L1, PC2, L2 successful on R1?**
> ✅ Yes — all 4 devices connected successfully.
> PAT allows unlimited simultaneous connections by using unique port numbers for each session.

**Q2: Were all connections from PC3, L3, PC4, L4 successful on R2?**
> ✅ Yes — all 4 devices connected successfully using R2's single interface IP.

**Q3: Why doesn't R2 list any dynamic mappings in statistics?**
> Because R2 uses **interface-based PAT** instead of a pool.
> When using `interface` instead of a pool name, the router uses its own interface IP directly.
> There is no pool to track — so the statistics show **0 dynamic pool mappings**.
> The translations still appear in `show ip nat translations` but are tracked differently.

---

## 🔄 PAT — How Port Numbers Enable Many-to-One

```
[PC1] 172.16.1.2  ──port 1025──→  209.165.200.233:1025  ──→  Server1
[L1]  172.16.1.3  ──port 1026──→  209.165.200.233:1026  ──→  Server1
[PC2] 172.16.2.2  ──port 1027──→  209.165.200.233:1027  ──→  Server1
[L2]  172.16.2.3  ──port 1028──→  209.165.200.233:1028  ──→  Server1

All using the SAME public IP — differentiated by PORT NUMBER only!
Theoretical max: 65,536 simultaneous connections per public IP
```

---

## 💡 Key Concepts Summary

| Concept | Details |
|---------|---------|
| **`overload` keyword** | Converts Dynamic NAT to PAT — enables port multiplexing |
| **PAT with Pool** | Multiple IPs in pool + overload — used on R1 |
| **PAT with Interface** | Uses router's own outside IP — no pool needed — used on R2 |
| **Port numbers** | What PAT uses to track which translation belongs to which host |
| **65,536 limit** | Max port numbers per IP — theoretical PAT capacity |
| **`show ip nat translations`** | Shows each session with its unique port mapping |
| **`clear ip nat translation *`** | Clears all dynamic/PAT translations |

---

## 📊 R1 vs R2 — PAT Method Comparison

| | R1 (PAT with Pool) | R2 (PAT with Interface) |
|--|-------------------|------------------------|
| **Command** | `ip nat inside source list 1 pool NAME overload` | `ip nat inside source list 2 interface s0/1/1 overload` |
| **Public IPs** | 2 IPs from pool | 1 IP (interface's own IP) |
| **Pool required** | ✅ Yes | ❌ No |
| **Best for** | When you have multiple public IPs | When you have only one public IP |
| **Statistics** | Shows dynamic pool mappings | Shows 0 dynamic pool mappings |

---

## 📝 What I Learned

- How PAT extends Dynamic NAT by adding the `overload` keyword
- Why PAT can handle thousands of devices with just one public IP address
- The two methods of configuring PAT — pool-based (R1) vs interface-based (R2)
- Why R2 shows no dynamic pool mappings in statistics — interface PAT works differently
- How port numbers are the key mechanism that makes PAT possible
- Why PAT is the most widely used form of NAT in real-world networks today
