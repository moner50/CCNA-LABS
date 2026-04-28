# 🛡️ Lab 5.1.9 — Configure Named Standard IPv4 ACLs

## 📌 Lab Info

| Field | Details |
|-------|---------|
| **Course** | CCNA — Network Security |
| **Lab** | 5.1.9 |
| **Tool** | Cisco Packet Tracer |
| **Difficulty** | ⭐⭐⭐☆☆ Intermediate |
| **Topic** | Named Standard IPv4 Access Control Lists |

> 📥 **Download the Packet Tracer file** and open it in Cisco Packet Tracer to test the configuration yourself using the ping tests below.

---

## 🎯 Objectives

### Part 1 — Configure and Apply a Named Standard ACL
- Create a named standard ACL to restrict access to the File Server
- Apply the ACL on the correct interface in the correct direction

### Part 2 — Verify the ACL Implementation
- Test connectivity using ping from each device
- Confirm only permitted devices can reach the File Server

---

## 🗺️ Topology Overview

> 📷 See `topology.png` for the full network diagram.

---

## 📋 Addressing Table

| Device | Interface | IP Address | Subnet Mask | Default Gateway |
|--------|-----------|------------|-------------|-----------------|
| R1 | F0/0 | 192.168.100.1 | 255.255.255.0 | N/A |
| R1 | F0/1 | 192.168.200.1 | 255.255.255.0 | N/A |
| R1 | E0/0/0 | 192.168.10.1 | 255.255.255.0 | N/A |
| R1 | E0/1/0 | 192.168.20.1 | 255.255.255.0 | N/A |
| File Server | NIC | 192.168.200.100 | 255.255.255.0 | 192.168.200.1 |
| Web Server | NIC | 192.168.100.100 | 255.255.255.0 | 192.168.100.1 |
| PC0 | NIC | 192.168.20.3 | 255.255.255.0 | 192.168.20.1 |
| PC1 (Web Manager) | NIC | 192.168.20.4 | 255.255.255.0 | 192.168.20.1 |
| PC2 | NIC | 192.168.10.3 | 255.255.255.0 | 192.168.10.1 |

---

## 📋 Network Policy

| Access | Device | To File Server (`192.168.200.100`) |
|--------|--------|-----------------------------------|
| ✅ Permit | **PC1** (Web Manager) `192.168.20.4` | Allowed |
| ✅ Permit | **Web Server** `192.168.100.100` | Allowed |
| ❌ Deny | **All other traffic** | Blocked |

---

## 🧠 ACL Planning

### Why Named ACL?
Named ACLs are more descriptive and easier to manage than numbered ACLs.
They also allow **deleting individual lines** without removing the entire ACL.

### Why Standard ACL?
The policy only restricts by **source IP address** — no port or protocol filtering needed.

### Where to place it?
> Standard ACLs go **closest to the destination** to avoid over-blocking.

| Interface | Direction | Reason |
|-----------|-----------|--------|
| **F0/1** (toward File Server `192.168.200.0/24`) | **Outbound** | Filters traffic just before it reaches the File Server |

---

## ⚙️ Configuration — Part 1

### Step 1 — Create the Named Standard ACL

```
R1(config)# ip access-list standard PROTECT-FILESERVER
R1(config-std-nacl)# permit host 192.168.20.4
R1(config-std-nacl)# permit host 192.168.100.100
R1(config-std-nacl)# deny any
```

> 💡 `permit host 192.168.20.4` = permits only PC1 (Web Manager)
> 💡 `permit host 192.168.100.100` = permits only the Web Server
> 💡 `deny any` = explicitly denies everything else (the implicit deny does this too, but writing it makes it visible in `show access-lists` with match counters)

---

### Step 2 — Apply the ACL to the Interface

```
R1(config)# interface F0/1
R1(config-if)# ip access-group PROTECT-FILESERVER out
```

---

## 🧪 Ping Tests — Part 2

Run these ping tests to verify the ACL is working correctly:

| # | From | To (File Server) | Expected | Reason |
|---|------|-----------------|----------|--------|
| 1 | **PC1** `192.168.20.4` | `192.168.200.100` | ✅ **Success** | PC1 is explicitly permitted |
| 2 | **Web Server** `192.168.100.100` | `192.168.200.100` | ✅ **Success** | Web Server is explicitly permitted |
| 3 | **PC0** `192.168.20.3` | `192.168.200.100` | ❌ **Fail** | Not in the permit list — denied |
| 4 | **PC2** `192.168.10.3` | `192.168.200.100` | ❌ **Fail** | Not in the permit list — denied |

---

## 🔍 Verification Commands

```bash
# View ACL statements and match counters
show access-lists

# Verify ACL is applied to correct interface and direction
show ip interface F0/1

# View the ACL in running config
show running-config | section access-list
```

### Expected output — `show access-lists`

```
Standard IP access list PROTECT-FILESERVER
    10 permit host 192.168.20.4    (X matches)
    20 permit host 192.168.100.100 (X matches)
    30 deny   any                  (X matches)
```

> 💡 Check the **match counters** — they increase each time traffic hits that ACE.
> If the deny counter is increasing, blocked traffic is being caught correctly ✅

---

## 💡 Named vs Numbered ACL — Key Differences

| Feature | Numbered ACL | Named ACL |
|---------|-------------|-----------|
| Identified by | Number (e.g., `1`) | Name (e.g., `PROTECT-FILESERVER`) |
| Delete a single line | ❌ Not possible | ✅ Possible |
| More descriptive | ❌ No | ✅ Yes |
| Functionally different | No — both work the same way |

---

## 💡 Key Concepts Summary

| Concept | Details |
|---------|---------|
| **Named standard ACL** | `ip access-list standard [NAME]` |
| **`permit host`** | Matches a single specific IP address (host wildcard `0.0.0.0`) |
| **`deny any`** | Explicitly denies all remaining traffic |
| **Outbound ACL** | Applied on the interface **facing the destination** |
| **Standard ACL placement** | Always closest to the **destination** |
| **Match counters** | Shown in `show access-lists` — confirm which rules are being hit |

---

## 📝 What I Learned

- The difference between named and numbered ACLs — and why named ACLs are preferred in production
- How to use `permit host` to match a single specific IP address
- Why the explicit `deny any` at the end is useful — it shows match counters in `show access-lists`
- How to verify ACL behavior using ping tests and match counters
- The importance of placing standard ACLs closest to the destination to minimize over-blocking
