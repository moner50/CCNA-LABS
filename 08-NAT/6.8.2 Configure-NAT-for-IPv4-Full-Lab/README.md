# 🌐 6.8.2Lab — Configure NAT for IPv4 (Full Lab)

## 📌 Lab Info

| Field | Details |
|-------|---------|
| **Course** | CCNA — Network Address Translation |
| **Tool** | Cisco Packet Tracer / Physical Lab |
| **Difficulty** | ⭐⭐⭐⭐⭐ Advanced |
| **Topic** | Dynamic NAT + PAT (Pool & Interface) + Static NAT |

> 📥 **Download the Packet Tracer file** and test the full configuration yourself.

---

## 🎯 Objectives

- **Part 1** — Build the network and configure basic device settings
- **Part 2** — Configure and verify Dynamic NAT
- **Part 3** — Configure and verify PAT (Pool & Interface)
- **Part 4** — Configure and verify Static NAT

---

## 🗺️ Topology Overview

> 📷 See `topology.png` for the full network diagram.
> 📷 See `score.png` for the lab result.

---

## 📋 Addressing Table

| Device | Interface | IP Address | Subnet Mask |
|--------|-----------|------------|-------------|
| R1 | G0/0/0 | 209.165.200.230 | 255.255.255.248 |
| R1 | G0/0/1 | 192.168.1.1 | 255.255.255.0 |
| R2 | G0/0/0 | 209.165.200.225 | 255.255.255.248 |
| R2 | Lo1 | 209.165.200.1 | 255.255.255.224 |
| S1 | VLAN 1 | 192.168.1.11 | 255.255.255.0 |
| S2 | VLAN 1 | 192.168.1.12 | 255.255.255.0 |
| PC-A | NIC | 192.168.1.2 | 255.255.255.0 |
| PC-B | NIC | 192.168.1.3 | 255.255.255.0 |

---

## 🔑 Public Address Space Planning

ISP allocated: `209.165.200.224/29`

| Address | Role |
|---------|------|
| `209.165.200.225` | R2 G0/0/0 — ISP side |
| `209.165.200.226` | NAT Pool — PC-B |
| `209.165.200.227` | NAT Pool — PC-A |
| `209.165.200.228` | NAT Pool — S1 |
| `209.165.200.229` | Static NAT — PC-A public address |
| `209.165.200.230` | R1 G0/0/0 — company gateway |

---

## ⚙️ Part 1 — Basic Device Configuration

### R1
```
Router# conf t
Router(config)# hostname R1
R1(config)# no ip domain-lookup
R1(config)# enable secret class
R1(config)# line console 0
R1(config-line)# password cisco
R1(config-line)# login
R1(config-line)# exit
R1(config)# line vty 0 4
R1(config-line)# password cisco
R1(config-line)# login
R1(config-line)# exit
R1(config)# service password-encryption
R1(config)# banner motd $ Unauthorized Access is Prohibited! $
R1(config)# interface g0/0/0
R1(config-if)# ip address 209.165.200.230 255.255.255.248
R1(config-if)# no shutdown
R1(config-if)# exit
R1(config)# interface g0/0/1
R1(config-if)# ip address 192.168.1.1 255.255.255.0
R1(config-if)# no shutdown
R1(config-if)# exit
R1(config)# ip route 0.0.0.0 0.0.0.0 209.165.200.225
R1# copy running-config startup-config
```

### R2
```
Router# conf t
Router(config)# hostname R2
R2(config)# no ip domain-lookup
R2(config)# enable secret class
R2(config)# line console 0
R2(config-line)# password cisco
R2(config-line)# login
R2(config-line)# exit
R2(config)# line vty 0 4
R2(config-line)# password cisco
R2(config-line)# login
R2(config-line)# exit
R2(config)# service password-encryption
R2(config)# banner motd $ Unauthorized Access is Prohibited! $
R2(config)# interface g0/0/0
R2(config-if)# ip address 209.165.200.225 255.255.255.248
R2(config-if)# no shutdown
R2(config-if)# exit
R2(config)# interface lo1
R2(config-if)# ip address 209.165.200.1 255.255.255.224
R2(config-if)# no shutdown
R2(config-if)# exit
R2# copy running-config startup-config
```

### S1
```
Switch# conf t
Switch(config)# hostname S1
S1(config)# no ip domain-lookup
S1(config)# enable secret class
S1(config)# line console 0
S1(config-line)# password cisco
S1(config-line)# login
S1(config-line)# exit
S1(config)# line vty 0 4
S1(config-line)# password cisco
S1(config-line)# login
S1(config-line)# exit
S1(config)# service password-encryption
S1(config)# banner motd $ Unauthorized Access is Prohibited! $
S1(config)# interface vlan 1
S1(config-if)# ip address 192.168.1.11 255.255.255.0
S1(config-if)# no shutdown
S1(config-if)# exit
S1(config)# ip default-gateway 192.168.1.1
S1# copy running-config startup-config
```

### S2
```
Switch# conf t
Switch(config)# hostname S2
S2(config)# no ip domain-lookup
S2(config)# enable secret class
S2(config)# line console 0
S2(config-line)# password cisco
S2(config-line)# login
S2(config-line)# exit
S2(config)# line vty 0 4
S2(config-line)# password cisco
S2(config-line)# login
S2(config-line)# exit
S2(config)# service password-encryption
S2(config)# banner motd $ Unauthorized Access is Prohibited! $
S2(config)# interface vlan 1
S2(config-if)# ip address 192.168.1.12 255.255.255.0
S2(config-if)# no shutdown
S2(config-if)# exit
S2(config)# ip default-gateway 192.168.1.1
S2# copy running-config startup-config
```

---

## ⚙️ Part 2 — Dynamic NAT Configuration

```
R1(config)# access-list 1 permit 192.168.1.0 0.0.0.255
R1(config)# ip nat pool PUBLIC_ACCESS 209.165.200.226 209.165.200.228 netmask 255.255.255.248
R1(config)# ip nat inside source list 1 pool PUBLIC_ACCESS
R1(config)# interface g0/0/1
R1(config-if)# ip nat inside
R1(config-if)# exit
R1(config)# interface g0/0/0
R1(config-if)# ip nat outside
R1(config-if)# exit
```

### Part 2 — Translation Table Results

| Device | Inside Local | Inside Global |
|--------|-------------|---------------|
| PC-B | 192.168.1.3 | 209.165.200.226 |
| PC-A | 192.168.1.2 | 209.165.200.227 |
| S1 | 192.168.1.11 | 209.165.200.228 |
| S2 | ❌ Failed | Pool exhausted — only 3 addresses! |

> ⚠️ Dynamic NAT is one-to-one — pool limited to 3 addresses means only 3 devices simultaneously

---

## ⚙️ Part 3 — PAT Configuration

### Step 1 — Remove Dynamic NAT, Add PAT with Pool
```
R1(config)# no ip nat inside source list 1 pool PUBLIC_ACCESS
R1(config)# ip nat inside source list 1 pool PUBLIC_ACCESS overload
```

### Step 2 — Remove Pool, Switch to PAT with Interface
```
R1(config)# no ip nat inside source list 1 pool PUBLIC_ACCESS overload
R1(config)# no ip nat pool PUBLIC_ACCESS
R1(config)# ip nat inside source list 1 interface g0/0/0 overload
```

### Part 3 — PAT Interface Translation Table

```
Pro  Inside global          Inside local       Outside local      Outside global
icmp 209.165.200.230:3      192.168.1.11:1     209.165.200.1:1    209.165.200.1:3
icmp 209.165.200.230:2      192.168.1.2:1      209.165.200.1:1    209.165.200.1:2
icmp 209.165.200.230:4      192.168.1.3:1      209.165.200.1:1    209.165.200.1:4
icmp 209.165.200.230:1      192.168.1.12:1     209.165.200.1:1    209.165.200.1:1
```

> All 4 devices share `209.165.200.230` — differentiated by **port numbers only** ✅

---

## ⚙️ Part 4 — Static NAT for PC-A

```
R1# clear ip nat translations *
R1# clear ip nat statistics
R1(config)# ip nat inside source static 192.168.1.2 209.165.200.229
```

### Part 4 — Static NAT Translation Table

```
Pro  Inside global       Inside local    Outside local      Outside global
---  209.165.200.229     192.168.1.2     ---                ---
icmp 209.165.200.229:3   192.168.1.2:3   209.165.200.225:3  209.165.200.225:3
```

---

## ❓ Lab Questions & Answers

**Q: What was PC-B's inside local address translated to in Dynamic NAT?**
> `209.165.200.226` — the first available address from the PUBLIC_ACCESS pool

**Q: What type of NAT address is the translated address?**
> **Inside Global** — the public IP representing the inside device after translation

**Q: What is different about PAT output vs NAT output?**
> PAT shows **port numbers** after each IP (e.g., `209.165.200.226:1`)
> Dynamic NAT shows **no port numbers** and creates static entries per device

**Q: How does the router track replies in PAT?**
> Using **unique port numbers** — each session gets a different source port.
> When a reply arrives, the router checks the destination port to know which inside host to forward it to

**Q: Why did S2 fail in Dynamic NAT?**
> The pool only had **3 addresses** (226, 227, 228) — all were used by PC-A, PC-B, and S1.
> PAT solves this by allowing all devices to share one IP via port multiplexing

---

## 🔍 Verification Commands

```bash
show ip nat translations          # View active translations
show ip nat translations verbose  # View translations with timeout info
show ip nat statistics            # View NAT counters and interface info
clear ip nat translations *       # Clear all dynamic translations
clear ip nat statistics           # Reset NAT counters
```

---

## 📊 All 3 NAT Types — Summary

| Type | Command | Addresses Used | Simultaneous Users |
|------|---------|---------------|-------------------|
| **Dynamic NAT** | `ip nat inside source list 1 pool NAME` | One IP per device | Limited by pool |
| **PAT Pool** | `ip nat inside source list 1 pool NAME overload` | Port multiplexing | Thousands |
| **PAT Interface** | `ip nat inside source list 1 interface g0/0/0 overload` | Router's own IP | Thousands |
| **Static NAT** | `ip nat inside source static [local] [global]` | Fixed one-to-one | One device |

---

## 💡 Key Concepts Summary

| Concept | Details |
|---------|---------|
| **Dynamic NAT timeout** | 24 hours — translations stay for a full day |
| **PAT timeout** | 1 minute — much shorter, port reused quickly |
| **Pool exhaustion** | Dynamic NAT fails when all pool IPs are in use |
| **Port numbers** | PAT's mechanism to track multiple sessions on one IP |
| **Static NAT** | Always in translation table — never times out |
| **`overload` keyword** | The single word that changes Dynamic NAT into PAT |

---

## 📝 What I Learned

- The practical limitations of Dynamic NAT — pool exhaustion with only 3 devices!
- How adding `overload` transforms NAT into PAT and solves the pool exhaustion problem
- Two ways to configure PAT — pool-based and interface-based
- Why Static NAT is essential for servers — permanent, predictable public address
- How PAT port numbers are the key to tracking thousands of simultaneous sessions
- The difference in timeout — 24 hours for NAT vs 1 minute for PAT
