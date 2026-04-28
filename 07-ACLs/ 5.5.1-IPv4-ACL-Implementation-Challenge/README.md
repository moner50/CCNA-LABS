# 🏆 Lab 5.5.1 — IPv4 ACL Implementation Challenge

## 📌 Lab Info

| Field | Details |
|-------|---------|
| **Course** | CCNA — Network Security |
| **Lab** | 5.5.1 |
| **Tool** | Cisco Packet Tracer |
| **Difficulty** | ⭐⭐⭐⭐⭐ Advanced |
| **Topic** | Full ACL Implementation — Standard, Extended & Named ACLs |

> 📥 **Download the Packet Tracer file** and test the full configuration yourself.

---

## 🎯 Objectives

- Configure a router with standard named ACLs
- Configure a router with extended named ACLs
- Configure extended ACLs to meet specific communication requirements
- Configure the correct interface with ACL in the correct direction
- Verify all ACLs using ping and FTP tests

---

## 🗺️ Topology Overview

> 📷 See `topology.png` for the full network diagram.
> 📷 See `score.png` for the lab score result.

---

## 📋 Addressing Table

| Device | Interface | IP Address |
|--------|-----------|------------|
| HQ | G0/0/0 | 192.168.1.1/26 |
| HQ | G0/0/1 | 192.168.1.65/29 |
| HQ | S0/1/0 | 192.0.2.1/30 |
| HQ | S0/1/1 | 192.168.3.1/30 |
| Branch | G0/0/0 | 192.168.2.1/27 |
| Branch | G0/0/1 | 192.168.2.33/28 |
| Branch | S0/1/1 | 192.168.3.2/30 |
| PC-1 | NIC | 192.168.1.10/26 |
| PC-2 | NIC | 192.168.1.20/26 |
| PC-3 | NIC | 192.168.1.30/26 |
| Admin | NIC | 192.168.1.67/29 |
| Enterprise Web Server | NIC | 192.168.1.70/29 |
| Branch PC | NIC | 192.168.2.17/27 |
| Branch Server | NIC | 192.168.2.45/28 |
| Internet User | NIC | 198.51.100.218/24 |
| External Web Server | NIC | 203.0.113.73/24 |

---

## 🌐 Network Summary

| Network | Subnet | Wildcard | Hosts |
|---------|--------|----------|-------|
| HQ LAN 1 | 192.168.1.0/26 | 0.0.0.63 | PC-1, PC-2, PC-3 |
| HQ LAN 2 | 192.168.1.64/29 | 0.0.0.7 | Admin, Enterprise Web Server |
| Branch LAN 1 | 192.168.2.0/27 | 0.0.0.31 | Branch PC |
| Branch LAN 2 | 192.168.2.32/28 | 0.0.0.15 | Branch Server |

---

## 📋 ACL Requirements & Policies

### ACL 101 — On HQ (Internet traffic filtering)

| Rule | Policy |
|------|--------|
| ❌ Deny | FTP from internet → Enterprise Web Server |
| ❌ Deny | ICMP from internet → HQ LAN 1 |
| ✅ Permit | All other traffic |

### branch_to_hq — On Branch (Branch to HQ filtering)

| Rule | Policy |
|------|--------|
| ❌ Deny | Branch LAN 1 → HQ LAN 1 |
| ❌ Deny | Branch LAN 2 → HQ LAN 1 |
| ✅ Permit | All other traffic |

---

## ⚙️ Full Configuration

---

### ACL 101 — HQ Router (Numbered Extended ACL)

```
HQ(config)# access-list 101 deny tcp any host 192.168.1.70 eq 21
HQ(config)# access-list 101 deny icmp any 192.168.1.0 0.0.0.63
HQ(config)# access-list 101 permit ip any any
```

**Apply inbound on S0/1/0 (internet-facing interface):**

```
HQ(config)# interface S0/1/0
HQ(config-if)# ip access-group 101 in
```

> 💡 Applied **inbound** on S0/1/0 because that is where internet traffic enters the HQ router.

---

### branch_to_hq — Branch Router (Named Extended ACL)

```
Branch(config)# ip access-list extended branch_to_hq
Branch(config-ext-nacl)# deny ip 192.168.2.0 0.0.0.31 192.168.1.0 0.0.0.63
Branch(config-ext-nacl)# deny ip 192.168.2.32 0.0.0.15 192.168.1.0 0.0.0.63
Branch(config-ext-nacl)# permit ip any any
```

**Apply outbound on S0/1/1 (link toward HQ):**

```
Branch(config)# interface S0/1/1
Branch(config-if)# ip access-group branch_to_hq out
```

> 💡 Applied **outbound** on S0/1/1 — all branch traffic to HQ passes through this single link,
> making it the most efficient point to filter both Branch LANs with one ACL.

---

## 🧪 Verification Tests

### ACL 101 Tests (from Internet)

| From | To | Protocol | Expected |
|------|----|----------|----------|
| Internet User `198.51.100.218` | Enterprise Web Server `192.168.1.70` | FTP | ❌ Blocked |
| Internet User `198.51.100.218` | PC-1 `192.168.1.10` | ICMP | ❌ Blocked |
| Internet User `198.51.100.218` | Enterprise Web Server `192.168.1.70` | HTTP | ✅ Allowed |
| Internet User `198.51.100.218` | Enterprise Web Server `192.168.1.70` | ICMP | ❌ Blocked (HQ LAN 1) |

### branch_to_hq Tests (from Branch)

| From | To | Expected |
|------|----|----------|
| Branch PC `192.168.2.17` | PC-1 `192.168.1.10` | ❌ Blocked |
| Branch PC `192.168.2.17` | PC-2 `192.168.1.20` | ❌ Blocked |
| Branch Server `192.168.2.45` | PC-3 `192.168.1.30` | ❌ Blocked |
| Branch PC `192.168.2.17` | External Web Server `203.0.113.73` | ✅ Allowed |
| Branch PC `192.168.2.17` | Enterprise Web Server `192.168.1.70` | ❌ Blocked (on HQ LAN 1) |

---

## 🔍 Verification Commands

```bash
# View all ACLs and match counters
show access-lists

# Confirm ACL applied to correct interface
show ip interface S0/1/0
show ip interface S0/1/1

# View ACL config in running config
show running-config | section access-list
```

---

## 💡 Key Concepts Summary

| Concept | Details |
|---------|---------|
| **ACL 101** | Numbered extended ACL — filters internet traffic entering HQ |
| **branch_to_hq** | Named extended ACL — name must match exactly (case-sensitive) |
| **Inbound on S0/1/0** | Stops unwanted internet traffic before it enters the HQ network |
| **Outbound on S0/1/1** | One ACL covers both Branch LANs heading to HQ |
| **No explicit deny any** | Lab guideline — rely on implicit deny instead |
| **`host` keyword** | Matches single IP — cleaner than `0.0.0.0` wildcard |
| **`any` keyword** | Matches all IPs — cleaner than `0.0.0.0 255.255.255.255` |
| **Wildcard /26** | `0.0.0.63` — covers all 64 hosts on HQ LAN 1 |
| **Wildcard /27** | `0.0.0.31` — covers all 32 hosts on Branch LAN 1 |
| **Wildcard /28** | `0.0.0.15` — covers all 16 hosts on Branch LAN 2 |

---

## 📝 What I Learned

- How to plan and implement multiple ACLs across different routers to meet specific business policies
- Why ACL placement and direction matter — wrong direction = wrong traffic filtered
- How one outbound ACL on a serial link can efficiently cover multiple source networks
- The difference between numbered extended (101) and named extended ACLs
- How to use `host` and `any` shorthand to write cleaner, more readable ACL statements
- Why the lab guidelines say no explicit `deny any` — trusting the implicit deny at the end
