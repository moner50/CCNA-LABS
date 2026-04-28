# 🛡️ Lab 5.4.13 — Configure Extended IPv4 ACLs (Scenario 2)

## 📌 Lab Info

| Field | Details |
|-------|---------|
| **Course** | CCNA — Network Security |
| **Lab** | 5.4.13 |
| **Tool** | Cisco Packet Tracer |
| **Difficulty** | ⭐⭐⭐⭐☆ Advanced |
| **Topic** | Named Extended IPv4 ACL — Per-Host Per-Protocol Filtering |

> 📥 **Download the Packet Tracer file** and test the configuration yourself using the tests below.

---

## 🎯 Objectives

### Part 1 — Configure a Named Extended ACL
- Deny HTTP and HTTPS from PC1 to Server1 and Server2
- Deny FTP from PC2 to Server1 and Server2
- Deny ICMP from PC3 to Server1 and Server2
- Permit all other traffic

### Part 2 — Apply and Verify the Extended ACL
- Apply the ACL to the correct interface and direction
- Test access for each PC and verify using match counters

---

## 🗺️ Topology Overview

> 📷 See `topology.png` for the full network diagram.

---

## 📋 Addressing Table

| Device | Interface | IP Address | Subnet Mask | Default Gateway |
|--------|-----------|------------|-------------|-----------------|
| RT1 | G0/0 | 172.31.1.126 | 255.255.255.224 | N/A |
| RT1 | S0/0/0 | 209.165.1.2 | 255.255.255.252 | N/A |
| PC1 | NIC | 172.31.1.101 | 255.255.255.224 | 172.31.1.126 |
| PC2 | NIC | 172.31.1.102 | 255.255.255.224 | 172.31.1.126 |
| PC3 | NIC | 172.31.1.103 | 255.255.255.224 | 172.31.1.126 |
| Server1 | NIC | 64.101.255.254 | — | — |
| Server2 | NIC | 64.103.255.254 | — | — |

---

## 📋 Network Policy

| Host | Protocol | Destination | Action |
|------|----------|-------------|--------|
| **PC1** | HTTP (port 80) | Server1 & Server2 | ❌ Deny |
| **PC1** | HTTPS (port 443) | Server1 & Server2 | ❌ Deny |
| **PC2** | FTP (port 21) | Server1 & Server2 | ❌ Deny |
| **PC3** | ICMP (ping) | Server1 & Server2 | ❌ Deny |
| **All** | Any other traffic | Anywhere | ✅ Permit |

---

## ⚙️ Full Configuration

### Part 1 — Create the Named Extended ACL

```
RT1# conf t

RT1(config)# ip access-list extended ACL

RT1(config-ext-nacl)# deny tcp host 172.31.1.101 host 64.101.255.254 eq 80
RT1(config-ext-nacl)# deny tcp host 172.31.1.101 host 64.101.255.254 eq 443
RT1(config-ext-nacl)# deny tcp host 172.31.1.101 host 64.103.255.254 eq 80
RT1(config-ext-nacl)# deny tcp host 172.31.1.101 host 64.103.255.254 eq 443

RT1(config-ext-nacl)# deny tcp host 172.31.1.102 host 64.101.255.254 eq 21
RT1(config-ext-nacl)# deny tcp host 172.31.1.102 host 64.103.255.254 eq 21

RT1(config-ext-nacl)# deny icmp host 172.31.1.103 host 64.101.255.254
RT1(config-ext-nacl)# deny icmp host 172.31.1.103 host 64.103.255.254

RT1(config-ext-nacl)# permit ip any any
RT1(config-ext-nacl)# exit
```

---

### Part 2 — Apply ACL to Interface

> Traffic originates from LAN `172.31.1.96/27` → apply on **G0/0 inbound** (closest to source)

```
RT1(config)# interface G0/0
RT1(config-if)# ip access-group ACL in
RT1(config-if)# end
```

---

## 🔍 Verify Before Applying

```
RT1# show access-lists
```

**Expected output:**
```
Extended IP access list ACL
    10 deny tcp host 172.31.1.101 host 64.101.255.254 eq www
    20 deny tcp host 172.31.1.101 host 64.101.255.254 eq 443
    30 deny tcp host 172.31.1.101 host 64.103.255.254 eq www
    40 deny tcp host 172.31.1.101 host 64.103.255.254 eq 443
    50 deny tcp host 172.31.1.102 host 64.101.255.254 eq ftp
    60 deny tcp host 172.31.1.102 host 64.103.255.254 eq ftp
    70 deny icmp host 172.31.1.103 host 64.101.255.254
    80 deny icmp host 172.31.1.103 host 64.103.255.254
    90 permit ip any any
```

---

## 🧪 Test Results — Part 2

### PC1 Tests
| Test | Destination | Expected |
|------|-------------|----------|
| HTTP browser | Server1 & Server2 | ❌ **Blocked** |
| HTTPS browser | Server1 & Server2 | ❌ **Blocked** |
| FTP | Server1 & Server2 | ✅ **Allowed** |
| Ping | Server1 & Server2 | ✅ **Allowed** |

### PC2 Tests
| Test | Destination | Expected |
|------|-------------|----------|
| FTP | Server1 & Server2 | ❌ **Blocked** |
| HTTP browser | Server1 & Server2 | ✅ **Allowed** |
| Ping | Server1 & Server2 | ✅ **Allowed** |

### PC3 Tests
| Test | Destination | Expected |
|------|-------------|----------|
| Ping | Server1 & Server2 | ❌ **Blocked** |
| HTTP browser | Server1 & Server2 | ✅ **Allowed** |
| FTP | Server1 & Server2 | ✅ **Allowed** |

---

## 🔍 Verification Commands

```bash
# View ACL with line numbers and match counters
show access-lists

# Confirm ACL applied to interface
show ip interface G0/0

# View ACL in running config
show running-config | begin access-list
```

### After running all tests — `show access-lists`

```
Extended IP access list ACL
    10 deny tcp host 172.31.1.101 host 64.101.255.254 eq www  (12 match(es))
    20 deny tcp host 172.31.1.101 host 64.101.255.254 eq 443  (12 match(es))
    30 deny tcp host 172.31.1.101 host 64.103.255.254 eq www
    40 deny tcp host 172.31.1.101 host 64.103.255.254 eq 443
    50 deny tcp host 172.31.1.102 host 64.101.255.254 eq ftp
    60 deny tcp host 172.31.1.102 host 64.103.255.254 eq ftp
    70 deny icmp host 172.31.1.103 host 64.101.255.254
    80 deny icmp host 172.31.1.103 host 64.103.255.254
    90 permit ip any any
```

> 💡 To reset match counters:
> ```
> RT1# clear access-list counters
> ```

---

## 💡 Key Concepts Summary

| Concept | Details |
|---------|---------|
| **Named extended ACL** | `ip access-list extended [NAME]` |
| **`host` keyword** | Matches a single specific IP — equivalent to `/32` wildcard `0.0.0.0` |
| **`eq 80`** | Matches HTTP traffic only |
| **`eq 443`** | Matches HTTPS traffic only |
| **`eq 21`** | Matches FTP control — port 21 only as specified |
| **`deny icmp`** | Blocks ping — no port needed for ICMP |
| **`permit ip any any`** | Allows all traffic not matched by deny rules |
| **Inbound on G0/0** | Extended ACL placed closest to the source LAN |
| **Match counters** | Shown in `show access-lists` — confirm rules are being triggered |
| **Sequence numbers** | Allow editing, inserting, and deleting individual ACEs |

---

## 📊 show access-lists vs show running-config

| Command | Shows Sequence Numbers | Shows Match Counters |
|---------|----------------------|---------------------|
| `show access-lists` | ✅ Yes | ✅ Yes |
| `show running-config` | ❌ No | ❌ No |

> 💡 Always use `show access-lists` when troubleshooting — it gives you both line numbers and match counters in one output.

---

## 📝 What I Learned

- How to configure a named extended ACL that filters **per host and per protocol**
- Why extended ACLs are placed **inbound on the interface closest to the source**
- The difference between `show access-lists` and `show running-config` for ACL output
- How sequence numbers enable **in-place editing** of individual ACEs
- How to use `clear access-list counters` to reset match counters for fresh testing
- Why each host can have completely different access restrictions in a single ACL
