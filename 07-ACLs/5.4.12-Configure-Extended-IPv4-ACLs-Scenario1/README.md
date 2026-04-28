# 🛡️ Lab 5.4.12 — Configure Extended IPv4 ACLs (Scenario 1)

## 📌 Lab Info

| Field | Details |
|-------|---------|
| **Course** | CCNA — Network Security |
| **Lab** | 5.4.12 |
| **Tool** | Cisco Packet Tracer |
| **Difficulty** | ⭐⭐⭐⭐☆ Advanced |
| **Topic** | Extended IPv4 ACLs — Numbered & Named |

> 📥 **Download the Packet Tracer file** and test the configuration yourself using the ping and FTP tests below.

---

## 🎯 Objectives

### Part 1 — Configure a Numbered Extended ACL
- Configure an ACL to permit **FTP** and **ICMP** from the PC1 LAN
- Apply the ACL on the correct interface to filter traffic
- Verify the ACL implementation

### Part 2 — Configure a Named Extended ACL
- Configure an ACL to permit **HTTP** and **ICMP** from the PC2 LAN
- Apply the ACL on the correct interface to filter traffic
- Verify the ACL implementation

---

## 🗺️ Topology Overview

> 📷 See `topology.png` for the full network diagram.

---

## 🧠 Extended ACL vs Standard ACL

| Feature | Standard ACL | Extended ACL |
|---------|-------------|--------------|
| Filters by | Source IP only | Source IP, Destination IP, Protocol, Port |
| Number range | 1–99, 1300–1999 | 100–199, 2000–2699 |
| Placement | Closest to **destination** | Closest to **source** |
| Granularity | Low | High ✅ |

> 💡 Extended ACLs go **closest to the source** — so unwanted traffic is blocked early and doesn't waste bandwidth traveling the network.

---

## 📋 Network Policies

### Part 1 — PC1 LAN Policy (Numbered Extended ACL)

| Protocol | From (PC1 LAN) | To | Action |
|----------|---------------|-----|--------|
| FTP (TCP port 21) | `PC1 LAN` | Any | ✅ Permit |
| ICMP (ping) | `PC1 LAN` | Any | ✅ Permit |
| All other traffic | `PC1 LAN` | Any | ❌ Deny |

### Part 2 — PC2 LAN Policy (Named Extended ACL)

| Protocol | From (PC2 LAN) | To | Action |
|----------|---------------|-----|--------|
| HTTP (TCP port 80) | `PC2 LAN` | Any | ✅ Permit |
| ICMP (ping) | `PC2 LAN` | Any | ✅ Permit |
| All other traffic | `PC2 LAN` | Any | ❌ Deny |

---

## ⚙️ Configuration

---

### Part 1 — Numbered Extended ACL (FTP + ICMP from PC1 LAN)

#### Step 1 — Create the ACL

```
R1(config)# access-list 100 permit tcp [PC1-network] [wildcard] any eq 21
R1(config)# access-list 100 permit tcp [PC1-network] [wildcard] any eq 20
R1(config)# access-list 100 permit icmp [PC1-network] [wildcard] any
R1(config)# access-list 100 deny ip any any
```

> 💡 FTP uses **two ports**:
> - Port **21** → FTP Control (commands)
> - Port **20** → FTP Data (file transfer)
> Both must be permitted for FTP to work correctly!

**Example with real addresses:**
```
R1(config)# access-list 100 permit tcp 172.16.10.0 0.0.0.255 any eq 21
R1(config)# access-list 100 permit tcp 172.16.10.0 0.0.0.255 any eq 20
R1(config)# access-list 100 permit icmp 172.16.10.0 0.0.0.255 any
R1(config)# access-list 100 deny ip any any
```

#### Step 2 — Apply to Interface (closest to PC1 LAN — inbound)

```
R1(config)# interface [interface-toward-PC1-LAN]
R1(config-if)# ip access-group 100 in
```

---

### Part 2 — Named Extended ACL (HTTP + ICMP from PC2 LAN)

#### Step 1 — Create the ACL

```
R1(config)# ip access-list extended PC2-LAN-POLICY
R1(config-ext-nacl)# permit tcp [PC2-network] [wildcard] any eq 80
R1(config-ext-nacl)# permit icmp [PC2-network] [wildcard] any
R1(config-ext-nacl)# deny ip any any
```

**Example with real addresses:**
```
R1(config)# ip access-list extended PC2-LAN-POLICY
R1(config-ext-nacl)# permit tcp 172.16.20.0 0.0.0.255 any eq 80
R1(config-ext-nacl)# permit icmp 172.16.20.0 0.0.0.255 any
R1(config-ext-nacl)# deny ip any any
```

#### Step 2 — Apply to Interface (closest to PC2 LAN — inbound)

```
R1(config)# interface [interface-toward-PC2-LAN]
R1(config-if)# ip access-group PC2-LAN-POLICY in
```

---

## 🧪 Verification Tests — Part 1 & 2

### Part 1 — PC1 LAN Tests

| # | Test | Expected | Reason |
|---|------|----------|--------|
| 1 | FTP from PC1 to server | ✅ **Success** | TCP port 21 & 20 permitted |
| 2 | Ping from PC1 to server | ✅ **Success** | ICMP permitted |
| 3 | HTTP from PC1 to server | ❌ **Fail** | TCP port 80 not in ACL |
| 4 | Telnet from PC1 to server | ❌ **Fail** | TCP port 23 not in ACL |

### Part 2 — PC2 LAN Tests

| # | Test | Expected | Reason |
|---|------|----------|--------|
| 1 | HTTP from PC2 to server | ✅ **Success** | TCP port 80 permitted |
| 2 | Ping from PC2 to server | ✅ **Success** | ICMP permitted |
| 3 | FTP from PC2 to server | ❌ **Fail** | TCP port 21 not in ACL |
| 4 | Telnet from PC2 to server | ❌ **Fail** | TCP port 23 not in ACL |

---

## 🔍 Verification Commands

```bash
# View ACL statements and match counters
show access-lists

# Verify ACL applied to correct interface and direction
show ip interface [interface]

# View ACL in running config
show running-config | section access-list
```

### Expected output — `show access-lists`

```
Extended IP access list 100
    10 permit tcp 172.16.10.0 0.0.0.255 any eq ftp-data  (X matches)
    20 permit tcp 172.16.10.0 0.0.0.255 any eq ftp        (X matches)
    30 permit icmp 172.16.10.0 0.0.0.255 any              (X matches)
    40 deny ip any any                                     (X matches)

Extended IP access list PC2-LAN-POLICY
    10 permit tcp 172.16.20.0 0.0.0.255 any eq www        (X matches)
    20 permit icmp 172.16.20.0 0.0.0.255 any              (X matches)
    30 deny ip any any                                     (X matches)
```

---

## 💡 Key Concepts Summary

| Concept | Details |
|---------|---------|
| **Extended ACL placement** | Closest to the **source** — blocks traffic before it enters the network |
| **Inbound direction** | `ip access-group [ACL] in` on the source-facing interface |
| **FTP ports** | Port **20** (data) + Port **21** (control) — both required |
| **HTTP port** | Port **80** — `eq 80` or `eq www` |
| **ICMP** | No port number — just `permit icmp [src] [dst]` |
| **`deny ip any any`** | Explicit final deny — shows match counters for blocked traffic |
| **Numbered extended** | Range 100–199 |
| **Named extended** | `ip access-list extended [NAME]` |

---

## 🔢 Common Port Numbers for ACLs

| Protocol | Port | Keyword |
|----------|------|---------|
| FTP Data | 20 | `ftp-data` |
| FTP Control | 21 | `ftp` |
| SSH | 22 | `ssh` |
| Telnet | 23 | `telnet` |
| SMTP | 25 | `smtp` |
| DNS | 53 | `domain` |
| HTTP | 80 | `www` |
| HTTPS | 443 | — |

---

## 📝 What I Learned

- How extended ACLs filter by both **source IP and protocol/port** — much more powerful than standard ACLs
- Why extended ACLs are placed **closest to the source** to stop traffic early
- That FTP requires **two ports** (20 and 21) — forgetting port 20 breaks file transfers
- How to configure both **numbered** (100) and **named** extended ACLs
- How to use port keywords like `eq ftp`, `eq www` instead of numbers
- How match counters in `show access-lists` confirm which rules are being triggered
