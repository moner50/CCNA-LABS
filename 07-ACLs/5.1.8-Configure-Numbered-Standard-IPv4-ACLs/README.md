# 🛡️ Lab 5.1.8 — Configure Numbered Standard IPv4 ACLs

## 📌 Lab Info

| Field | Details |
|-------|---------|
| **Course** | CCNA — Network Security |
| **Lab** | 5.1.8 |
| **Tool** | Cisco Packet Tracer |
| **Difficulty** | ⭐⭐⭐☆☆ Intermediate |
| **Topic** | Numbered Standard IPv4 Access Control Lists |

> 📥 **Download the Packet Tracer file** and open it in Cisco Packet Tracer to follow along and test the configuration yourself.

---

## 🎯 Objectives

- Evaluate network policies and plan ACL implementations
- Configure numbered standard IPv4 ACLs on R2 and R3
- Apply ACLs to the correct interface in the correct direction
- Verify ACL behavior using ping tests

---

## 🗺️ Topology Overview

> 📷 See `topology.png` for the full network diagram.

| Network | Description |
|---------|-------------|
| `192.168.10.0/24` | PC1 LAN (connected to R3) |
| `192.168.11.0/24` | PC2 LAN (connected to R2) |
| `192.168.20.0/24` | WebServer LAN (connected to R2) — WebServer: `192.168.20.254` |
| `192.168.30.0/24` | PC3 LAN (connected to R3) |

---

## 📋 Network Policies

### 🔴 R2 Policies
| Policy | Details |
|--------|---------|
| ❌ Deny | `192.168.11.0/24` → WebServer (`192.168.20.254`) |
| ✅ Permit | All other traffic |

### 🔴 R3 Policies
| Policy | Details |
|--------|---------|
| ❌ Deny | `192.168.10.0/24` → `192.168.30.0/24` |
| ✅ Permit | All other traffic |

---

## 🧠 ACL Planning

### Why Standard ACL?
Standard ACLs filter based on **source IP address only**.
Since the policies only specify which **source network** to block, standard ACLs are sufficient.

### Where to place the ACL?
> Standard ACLs should be placed **closest to the destination** to avoid blocking more traffic than intended.

| Router | Interface | Direction | Reason |
|--------|-----------|-----------|--------|
| **R2** | Interface toward WebServer (`192.168.20.0/24`) | **Outbound** | Blocks traffic just before it reaches the WebServer |
| **R3** | Interface toward PC3 (`192.168.30.0/24`) | **Outbound** | Blocks traffic just before it reaches the 192.168.30.0 network |

---

## ⚙️ Configuration

### 🔧 R2 — ACL Configuration

```
R2(config)# access-list 1 deny 192.168.11.0 0.0.0.255
R2(config)# access-list 1 permit any
```

Apply to the outbound interface toward WebServer:
```
R2(config)# interface [interface-toward-192.168.20.0]
R2(config-if)# ip access-group 1 out
```

---

### 🔧 R3 — ACL Configuration

```
R3(config)# access-list 1 deny 192.168.10.0 0.0.0.255
R3(config)# access-list 1 permit any
```

Apply to the outbound interface toward PC3:
```
R3(config)# interface [interface-toward-192.168.30.0]
R3(config-if)# ip access-group 1 out
```

---

## ✅ Verification — Ping Tests

| Test | From | To | Expected | Result |
|------|------|----|----------|--------|
| 1 | `192.168.10.10` (PC1) | `192.168.11.10` (PC2) | ✅ Success | Neither policy blocks this |
| 2 | `192.168.10.10` (PC1) | `192.168.20.254` (WebServer) | ✅ Success | Only `192.168.11.0` is blocked from WebServer |
| 3 | `192.168.11.10` (PC2) | `192.168.20.254` (WebServer) | ❌ Fail | R2 ACL blocks `192.168.11.0` → WebServer |
| 4 | `192.168.10.10` (PC1) | `192.168.30.10` (PC3) | ❌ Fail | R3 ACL blocks `192.168.10.0` → `192.168.30.0` |
| 5 | `192.168.11.10` (PC2) | `192.168.30.10` (PC3) | ✅ Success | Only `192.168.10.0` is blocked from `192.168.30.0` |
| 6 | `192.168.30.10` (PC3) | `192.168.20.254` (WebServer) | ✅ Success | No policy restricts this traffic |

---

## 🔍 Verification Commands

```bash
# View ACL statements and match counters
show access-lists

# Verify ACL is applied to the correct interface and direction
show ip interface [interface]

# View running config ACL section
show running-config | include access-list
show running-config | include access-group
```

### Expected output — `show access-lists` on R2
```
Standard IP access list 1
    10 deny   192.168.11.0 0.0.0.255  (X matches)
    20 permit any                      (X matches)
```

> 💡 The **match counters** increase every time traffic hits that ACE — very useful for troubleshooting!

---

## 💡 Key Concepts Summary

| Concept | Details |
|---------|---------|
| **Standard ACL** | Filters by **source IP only** — numbered 1–99 and 1300–1999 |
| **Numbered ACL** | Identified by a number instead of a name |
| **Implicit deny** | Every ACL ends with an invisible `deny any` — always add `permit any` if needed |
| **Outbound ACL** | Applied on the interface facing the **destination** network |
| **Standard ACL placement** | Always place **closest to the destination** |
| **Wildcard mask** | `0.0.0.255` matches any host in a `/24` network |

---

## 📝 What I Learned

- How to read a network policy and translate it into ACL statements
- Why standard ACLs must be placed closest to the destination — to avoid over-blocking
- How a single `permit any` at the end prevents the implicit deny from blocking all other traffic
- How to verify ACL behavior using ping tests and `show access-lists`
- How match counters in `show access-lists` help confirm which ACEs are being hit
