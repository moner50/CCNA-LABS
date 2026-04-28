# 🛡️ Lab 5.2.7 — Configure and Modify Standard IPv4 ACLs

## 📌 Lab Info

| Field | Details |
|-------|---------|
| **Course** | CCNA — Network Security |
| **Lab** | 5.2.7 |
| **Tool** | Cisco Packet Tracer |
| **Difficulty** | ⭐⭐⭐⭐☆ Intermediate–Advanced |
| **Topic** | Named Standard ACL Configuration & In-Place Modification |

> 📥 **Download the Packet Tracer file** and test the configuration yourself using the ping tests below.

---

## 🎯 Objectives

- **Part 1** — Configure a named standard ACL (`BRANCH-OFFICEPOLICY`)
- **Part 2** — Verify ACL behavior with connectivity tests
- **Part 3** — Modify an existing ACL in place to match a new management policy

---

## 🗺️ Topology Overview

> 📷 See `topology.png` for the full network diagram.

| Network | Description |
|---------|-------------|
| `192.168.10.0/24` | Main LAN — destination network being protected |
| `192.168.40.0/24` | Branch office network |
| `209.165.200.224/27` | Internet — Web Server at `209.165.200.254` |
| **PC-A** | Host on `192.168.10.0/24` |
| **PC-C** | Specific host allowed from another network |

---

## 📋 Network Policies

### Part 1 — Initial Policy (BRANCH-OFFICEPOLICY)

| Access | Source | To `192.168.10.0/24` |
|--------|--------|----------------------|
| ✅ Permit | All hosts on `192.168.40.0/24` | Allowed |
| ✅ Permit | **PC-C only** (specific host) | Allowed |
| ❌ Deny | All other traffic | Blocked |

### Part 3 — Updated Policy (after management change)

| New Rule | Details |
|----------|---------|
| ✅ Also permit | Return traffic from `209.165.200.224/27` (Internet) |
| ➕ Add | Explicit `deny any` at the end of ALL ACLs |

---

## ⚙️ Configuration — Part 1

### Create the Named Standard ACL

```
R1(config)# ip access-list standard BRANCH-OFFICEPOLICY
R1(config-std-nacl)# permit 192.168.40.0 0.0.0.255
R1(config-std-nacl)# permit host [PC-C IP address]
R1(config-std-nacl)# deny any
```

### Apply to the Interface (outbound toward `192.168.10.0/24`)

```
R1(config)# interface G0/1
R1(config-if)# ip access-group BRANCH-OFFICEPOLICY out
```

---

## ✏️ Modification — Part 3

### Why modify?

After testing, a ping from PC-A to the Internet server (`209.165.200.254`) **fails** on the return trip. The returning packets come from `209.165.200.224/27` — which is not in the permit list, so the implicit deny blocks them.

Management also wants a consistent `deny any` at the end of all ACLs.

---

### Option 1 — Remove and Retype (not recommended)

```
R1(config)# no ip access-list standard BRANCH-OFFICEPOLICY
```
Then retype the entire ACL from scratch.

> ⚠️ Risk: Easy to make typos. If the ACL was long, this is very error-prone.

---

### Option 2 — Modify In Place ✅ (recommended)

View current ACL with line numbers first:

```
R1# show access-lists
Standard IP access list BRANCH-OFFICEPOLICY
    10 permit 192.168.40.0 0.0.0.255
    20 permit host [PC-C IP]
    30 deny   any
```

Add the new permit rule and explicit deny any:

```
R1(config)# ip access-list standard BRANCH-OFFICEPOLICY
R1(config-std-nacl)# no 30
R1(config-std-nacl)# permit 209.165.200.224 0.0.0.31
R1(config-std-nacl)# deny any
```

**Final ACL after modification:**

```
Standard IP access list BRANCH-OFFICEPOLICY
    10 permit 192.168.40.0 0.0.0.255
    20 permit host [PC-C IP]
    30 permit 209.165.200.224 0.0.0.31
    40 deny   any
```

> 💡 Line numbers make in-place editing easy — you can delete, insert, or replace specific lines without touching the rest of the ACL.

---

## 🧪 Ping Tests — Verify ACL Behavior

| # | From | To | Expected | Reason |
|---|------|----|----------|--------|
| 1 | Any host on `192.168.40.0/24` | `192.168.10.0/24` | ✅ **Success** | Entire subnet is permitted |
| 2 | **PC-C** | `192.168.10.0/24` | ✅ **Success** | PC-C is explicitly permitted |
| 3 | Any other host | `192.168.10.0/24` | ❌ **Fail** | Hits the `deny any` |
| 4 | **PC-A** | `209.165.200.254` (Internet) | ❌ **Fail** (before modification) | Return traffic from `209.165.200.224/27` was denied |
| 5 | **PC-A** | `209.165.200.254` (Internet) | ✅ **Success** (after modification) | Return traffic now permitted |

---

## 🔍 Verification Commands

```bash
# View ACL with line numbers and match counters
show access-lists

# Check ACL applied to interface
show ip interface G0/1

# View ACL in running config
show running-config | section access-list
```

### Expected output — `show access-lists` (after modification)

```
Standard IP access list BRANCH-OFFICEPOLICY
    10 permit 192.168.40.0, wildcard bits 0.0.0.255  (X matches)
    20 permit host [PC-C IP]                          (X matches)
    30 permit 209.165.200.224, wildcard bits 0.0.0.31 (X matches)
    40 deny   any                                      (X matches)
```

---

## 💡 Option 1 vs Option 2 — Comparison

| | Option 1 (Remove & Retype) | Option 2 (Modify In Place) ✅ |
|--|---------------------------|-------------------------------|
| **Method** | `no ip access-list standard [NAME]` then retype | Enter ACL config mode, delete/add specific lines |
| **Risk** | High — easy to introduce errors | Low — only touches what needs changing |
| **Best for** | Very short ACLs | Long ACLs or minor changes |
| **Production use** | Not recommended | Recommended |

---

## 💡 Key Concepts Summary

| Concept | Details |
|---------|---------|
| **Named standard ACL** | `ip access-list standard [NAME]` |
| **In-place modification** | Enter ACL config mode → use line numbers to add/delete specific ACEs |
| **`no [line number]`** | Removes a specific ACE without deleting the whole ACL |
| **Wildcard for /27** | `209.165.200.224 0.0.0.31` — 32-1=31 |
| **Explicit `deny any`** | Shows match counters — best practice for visibility |
| **Consistent ACL rules** | Management policy: all ACLs end with explicit `deny any` |

---

## 📝 What I Learned

- How to create a named standard ACL with multiple permit rules and a final deny
- Why Internet return traffic can be blocked by an ACL — and how to fix it
- The difference between Option 1 (remove & retype) and Option 2 (in-place modification)
- How to use ACL line numbers to delete and insert specific ACEs
- Why Option 2 (in-place editing) is the preferred method in real production networks
- The importance of consistent ACL policies across all routers in a network
