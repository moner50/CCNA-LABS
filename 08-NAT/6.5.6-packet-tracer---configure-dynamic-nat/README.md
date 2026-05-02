# 🌐 Lab 6.5.6 — Configure Dynamic NAT

## 📌 Lab Info

| Field | Details |
|-------|---------|
| **Course** | CCNA — Network Address Translation |
| **Lab** | 6.5.6 |
| **Tool** | Cisco Packet Tracer |
| **Difficulty** | ⭐⭐⭐☆☆ Intermediate |
| **Topic** | Dynamic NAT — Pool + ACL + Interface Configuration |

> 📥 **Download the Packet Tracer file** and test the configuration yourself.

---

## 🎯 Objectives

### Part 1 — Configure Dynamic NAT
- Define a NAT pool of public addresses
- Configure an ACL to permit inside traffic
- Bind the ACL to the NAT pool
- Configure NAT interfaces (inside & outside)

### Part 2 — Verify NAT Implementation
- Access a web server from inside hosts
- View and interpret the NAT translation table

---

## 🗺️ Topology Overview

> 📷 See `topology.png` for the full network diagram.

The topology has:
- **L1, PC1, PC2** — Inside hosts on `172.16.0.0/16`
- **R2** — NAT router
- **Server1** — Outside web server on the internet
- **NAT Pool** — `209.165.200.228/30` (only 2 usable addresses)

---

## 🧠 Static NAT vs Dynamic NAT

| Feature | Static NAT | Dynamic NAT |
|---------|-----------|-------------|
| Mapping | **Fixed** — one-to-one permanent | **Dynamic** — assigned from a pool |
| Direction | Works both ways | Inside → Outside only |
| Best for | Servers needing consistent public IP | Clients that need temporary internet access |
| Pool needed | ❌ No | ✅ Yes |

---

## ⚙️ Configuration — Part 1 (4 Steps)

---

### 🔧 Step 1 — Configure ACL to Permit Inside Traffic

Create a standard ACL to permit the inside network:

```
R2(config)# access-list 1 permit 172.16.0.0 0.0.255.255
```

> 💡 This tells NAT: *"Only translate traffic coming from the `172.16.0.0/16` network"*
> Wildcard `0.0.255.255` covers the entire `/16` — all hosts from `172.16.0.1` to `172.16.255.254`

---

### 🔧 Step 2 — Define the NAT Pool

```
R2(config)# ip nat pool NAT_POOL 209.165.200.229 209.165.200.230 netmask 255.255.255.252
```

> The `/30` network `209.165.200.228` gives us:
> - Network:    `209.165.200.228`
> - Usable IPs: `209.165.200.229` and `209.165.200.230` ← these are our 2 pool addresses
> - Broadcast:  `209.165.200.231`

---

### 🔧 Step 3 — Bind ACL to the Pool

```
R2(config)# ip nat inside source list 1 pool NAT_POOL
```

> This links everything together:
> *"Take traffic permitted by ACL 1 and translate it using addresses from NAT_POOL"*

---

### 🔧 Steps 4 & 5 — Configure NAT Interfaces

Mark **inside** interface (facing private network):
```
R2(config)# interface [inside-interface]
R2(config-if)# ip nat inside
R2(config-if)# exit
```

Mark **outside** interface (facing internet):
```
R2(config)# interface [outside-interface]
R2(config-if)# ip nat outside
R2(config-if)# exit
```

**Full example:**
```
R2(config)# interface g0/0
R2(config-if)# ip nat inside
R2(config-if)# exit

R2(config)# interface s0/0/1
R2(config-if)# ip nat outside
R2(config-if)# exit
```

---

## ❓ Important Question from the Lab

**Q: What will happen if more than 2 devices attempt to access the internet at the same time?**

> The **3rd device will be blocked** and will not get a translation.
> Dynamic NAT works on a **first-come, first-served** basis.
> The pool only has **2 addresses** — once both are in use, any new connection attempt
> will fail until one of the existing translations times out and that address is returned to the pool.
>
> This is a key limitation of Dynamic NAT — the pool size must match the expected number
> of simultaneous users. This is why **PAT (NAT Overload)** was invented — it allows
> thousands of devices to share a single public IP using port numbers.

---

## ✅ Verification — Part 2

### Step 1 — Access web page from inside hosts
Open the web browser on **L1, PC1, or PC2** and navigate to Server1's IP.
The page should load successfully ✅

---

### Step 2 — View NAT translations on R2

```
R2# show ip nat translations
```

**Expected output:**
```
Pro  Inside global       Inside local        Outside local       Outside global
tcp  209.165.200.229:XXXX 172.16.1.2:XXXX   209.165.200.226:80  209.165.200.226:80
tcp  209.165.200.230:XXXX 172.16.1.3:XXXX   209.165.200.226:80  209.165.200.226:80
```

| Column | Meaning |
|--------|---------|
| **Inside local** | Original private IP of the inside host |
| **Inside global** | Public IP assigned from the NAT pool |
| **Outside global** | IP of the destination server on the internet |

---

### Check NAT statistics

```
R2# show ip nat statistics
```

**Look for:**
```
Total active translations: 2 (0 static, 2 dynamic; 2 extended)
Peak translations: 2
Inside interfaces:  GigabitEthernet0/0
Outside interfaces: Serial0/0/1
Hits: XX  Misses: XX
```

---

## 🔄 How Dynamic NAT Works — Step by Step

```
[PC1]                    [R2 - NAT Router]              [Server1]
172.16.1.2   ─────────→  Checks ACL 1:                   209.165.200.226
             ─────────→  172.16.1.2 is permitted ✅
             ─────────→  Assigns 209.165.200.229 from pool
             ─────────→  Creates translation entry
                         ──────────────────────────────→  receives from 209.165.200.229

[PC2]                    [R2 - NAT Router]
172.16.1.3   ─────────→  Assigns 209.165.200.230 from pool ✅

[L1]                     [R2 - NAT Router]
172.16.1.4   ─────────→  Pool is FULL ❌ — connection fails
```

---

## 💡 Key Concepts Summary

| Concept | Details |
|---------|---------|
| **Dynamic NAT** | Assigns public IPs from a pool dynamically — first come, first served |
| **NAT Pool** | Range of public IPs available for translation |
| **Standard ACL** | Defines which inside hosts are allowed to be translated |
| **`ip nat inside source list`** | Binds the ACL to the NAT pool |
| **Pool exhaustion** | When all pool IPs are in use — new connections are denied |
| **Translation timeout** | Dynamic entries expire after idle time — returns IP to pool |
| **`show ip nat translations`** | Shows all active NAT mappings |
| **`clear ip nat translation *`** | Clears all dynamic NAT entries |

---

## 📝 What I Learned

- How Dynamic NAT differs from Static NAT — pool-based vs fixed mapping
- The 4-step process: ACL → Pool → Bind → Interfaces
- Why pool size is critical — more simultaneous users than pool IPs = connection failures
- How to read the NAT translation table and identify inside local vs inside global addresses
- Why Dynamic NAT led to the invention of PAT — to solve the pool exhaustion problem
- How to use `show ip nat statistics` to monitor NAT activity and troubleshoot issues
