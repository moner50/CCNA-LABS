# 🌐 Lab 6.4.5 — Configure Static NAT

## 📌 Lab Info

| Field | Details |
|-------|---------|
| **Course** | CCNA — Network Address Translation |
| **Lab** | 6.4.5 |
| **Tool** | Cisco Packet Tracer |
| **Difficulty** | ⭐⭐⭐☆☆ Intermediate |
| **Topic** | Static NAT Configuration & Verification |

> 📥 **Download the Packet Tracer file** and test the configuration yourself.

---

## 🎯 Objectives

### Part 1 — Test Access Without NAT
- Verify that outside hosts cannot reach inside servers without NAT

### Part 2 — Configure Static NAT
- Configure inside and outside NAT interfaces
- Create a static NAT mapping

### Part 3 — Test Access With NAT
- Verify outside hosts can now reach the inside server
- View and interpret NAT translation table

---

## 🗺️ Topology Overview

> 📷 See `topology.png` for the full network diagram.

---

## 🧠 NAT Concepts Review

### What is Static NAT?
Static NAT creates a **one-to-one permanent mapping** between a private (inside local) IP address and a public (inside global) IP address. It never changes — the same private IP always maps to the same public IP.

### The 4 NAT Address Types

| Term | Meaning | Example |
|------|---------|---------|
| **Inside Local** | Private IP of inside device — **before** translation | `192.168.1.10` |
| **Inside Global** | Public IP representing inside device — **after** translation | `209.165.200.5` |
| **Outside Global** | Real IP of the outside device | `8.8.8.8` |
| **Outside Local** | IP of outside device as seen from inside | Usually same as Outside Global |

---

## ⚙️ Configuration — Part 2

### Step 1 — Configure NAT Interfaces

Mark the **inside** interface (facing private network):
```
Router(config)# interface [inside-interface]
Router(config-if)# ip nat inside
Router(config-if)# exit
```

Mark the **outside** interface (facing public/internet):
```
Router(config)# interface [outside-interface]
Router(config-if)# ip nat outside
Router(config-if)# exit
```

**Example:**
```
Router(config)# interface g0/0
Router(config-if)# ip nat inside
Router(config-if)# exit

Router(config)# interface s0/0/0
Router(config-if)# ip nat outside
Router(config-if)# exit
```

---

### Step 2 — Create Static NAT Mapping

```
Router(config)# ip nat inside source static [inside-local-IP] [inside-global-IP]
```

**Example — map Server1's private IP to a public IP:**
```
Router(config)# ip nat inside source static 192.168.1.10 209.165.200.254
```

> This means: *"Anyone from outside trying to reach `209.165.200.254` will be forwarded to `192.168.1.10` inside the network."*

---

## ✅ Verification — Part 3

### Check NAT translations table
```
show ip nat translations
```

**Expected output:**
```
Pro  Inside global       Inside local        Outside local       Outside global
---  209.165.200.254     192.168.1.10        ---                 ---
tcp  209.165.200.254:80  192.168.1.10:80     X.X.X.X:YYYY        X.X.X.X:YYYY
```

> 💡 The first line shows the **static mapping** (always present).
> The second line appears when **active traffic** is flowing through NAT.

---

### Check NAT statistics
```
show ip nat statistics
```

**Look for:**
```
Total active translations: 1 (1 static, 0 dynamic; 0 extended)
Inside interfaces:  GigabitEthernet0/0
Outside interfaces: Serial0/0/0
```

---

### Verify interface NAT assignment
```
show ip interface [interface]
```

**Look for:**
```
Inbound access list is not set
Outbound access list is not set
Proxy ARP is enabled
...
NAT: Inside/Outside
```

---

## 🧪 Test Results

| Test | Before NAT | After NAT |
|------|-----------|-----------|
| Outside host pings `209.165.200.254` | ❌ Fails | ✅ Success |
| Outside host accesses Server1 web page | ❌ Fails | ✅ Success |
| Inside host accesses Server1 directly | ✅ Works | ✅ Works |

---

## 💡 Key Concepts Summary

| Concept | Details |
|---------|---------|
| **Static NAT** | One-to-one permanent mapping — same private IP always gets same public IP |
| **`ip nat inside`** | Marks the interface facing the **private network** |
| **`ip nat outside`** | Marks the interface facing the **public network / internet** |
| **`ip nat inside source static`** | Creates the permanent mapping between local and global IP |
| **`show ip nat translations`** | Shows active and static NAT mappings |
| **`show ip nat statistics`** | Shows NAT counters and interface assignments |
| **Inside Local** | Private IP before translation |
| **Inside Global** | Public IP after translation |

---

## 🔄 How Static NAT Works — Step by Step

```
[Outside Host]                [Router/NAT]              [Server1 Inside]
Sends packet to:              Checks NAT table:          Receives packet as:
209.165.200.254    ────────→  translates to         →    192.168.1.10
(Inside Global)               192.168.1.10               (Inside Local)

Server1 replies:              Translates back:           Outside host receives:
192.168.1.10       ────────→  translates to         →    209.165.200.254
(Inside Local)                209.165.200.254            (Inside Global)
```

---

## 📝 What I Learned

- How Static NAT creates a permanent one-to-one mapping between private and public IPs
- Why inside and outside interface designations are critical — NAT won't work without them
- How to read the NAT translation table and understand each column
- The difference between static entries (always present) and dynamic entries (appear during active sessions)
- Why Static NAT is used for servers that need to be consistently reachable from the internet
