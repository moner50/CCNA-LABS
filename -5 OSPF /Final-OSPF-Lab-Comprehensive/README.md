# 🏆 Final OSPF Lab — Comprehensive OSPFv2 Configuration

## 📌 Lab Info

| Field | Details |
|-------|---------|
| **Course** | CCNA — Routing & Switching |
| **Tool** | Cisco Packet Tracer |
| **Difficulty** | ⭐⭐⭐⭐⭐ Advanced |
| **Topic** | Full OSPFv2 Design & Configuration (Final Lab) |

---

## 🎯 Lab Requirements

| # | Requirement | Status |
|---|-------------|--------|
| 1 | Use process ID **10** for OSPF on all routers | ✅ |
| 2 | Activate OSPF using **network statements** on Headquarters routers | ✅ |
| 3 | Activate OSPF by **configuring interfaces** on Data Service network devices | ✅ |
| 4 | Configure **Router IDs** on multiaccess network routers | ✅ |
| 5 | Configure **passive interfaces** where routing updates are not required | ✅ |
| 6 | Configure **BC-1** with highest OSPF priority to always be DR | ✅ |
| 7 | Configure a **default route** to ISP using exit interface argument | ✅ |
| 8 | **Propagate default route** automatically to all routers | ✅ |
| 9 | Set **auto-cost reference-bandwidth** so GigabitEthernet=10, FastEthernet=100 | ✅ |
| 10 | Set **OSPF cost of P2P-1 Serial0/1/1** to 50 | ✅ |
| 11 | Configure **Hello & Dead timers** on P2P-1 ↔ BC-1 interfaces to twice the default | ✅ |

---

## 🗺️ Topology

> 📷 See `topology.png` for the full network diagram.

The topology is divided into two networks:
- **Headquarters Network** — OSPF activated via `network` statements
- **Data Service Network** — OSPF activated directly on interfaces
- **Multiaccess Network** — BC-1, BC-2, BC-3 with DR/BDR election
- **P2P-1** — Connected to both the HQ and Branch networks via serial links

---

## ⚙️ Full Configuration

---

### 1️⃣ OSPF Process ID 10 — All Routers

All routers use **process ID 10**:
```
Router(config)# router ospf 10
```

---

### 2️⃣ Headquarters Network — OSPF via Network Statements

```
HQ-Router(config)# router ospf 10
HQ-Router(config-router)# network [network-address] [wildcard-mask] area 0
```

**Example:**
```
HQ-Router(config-router)# network 192.168.1.0 0.0.0.255 area 0
HQ-Router(config-router)# network 10.0.0.0 0.0.0.3 area 0
```

---

### 3️⃣ Data Service Network — OSPF via Interface Configuration

```
Router(config)# interface [interface]
Router(config-if)# ip ospf 10 area 0
```

**Example:**
```
DS-Router(config)# interface GigabitEthernet0/0
DS-Router(config-if)# ip ospf 10 area 0
```

---

### 4️⃣ Router IDs on Multiaccess Network

```
BC-1(config)# router ospf 10
BC-1(config-router)# router-id 6.6.6.6

BC-2(config)# router ospf 10
BC-2(config-router)# router-id 5.5.5.5

BC-3(config)# router ospf 10
BC-3(config-router)# router-id 4.4.4.4
```

> After changing the Router ID, reset the OSPF process:
> ```
> Router# clear ip ospf process
> ```

---

### 5️⃣ Passive Interfaces

Apply on interfaces connected to end devices (no OSPF neighbors):

```
Router(config)# router ospf 10
Router(config-router)# passive-interface [interface]
```

> This stops Hello packets from being sent out LAN-facing interfaces
> while still advertising those networks into OSPF.

---

### 6️⃣ BC-1 — Highest OSPF Priority (Always DR)

Set priority to **255** on BC-1's multiaccess interface:

```
BC-1(config)# interface [multiaccess-interface]
BC-1(config-if)# ip ospf priority 255
```

> Priority range: 0–255. Default is 1.
> Priority 255 guarantees BC-1 always wins the DR election.
> Priority 0 = never becomes DR or BDR.

---

### 7️⃣ Default Route to ISP — Using Exit Interface

```
EdgeRouter(config)# ip route 0.0.0.0 0.0.0.0 [exit-interface]
```

**Example:**
```
EdgeRouter(config)# ip route 0.0.0.0 0.0.0.0 GigabitEthernet0/1
```

> Using the **exit interface** instead of a next-hop IP address is the requirement here.

---

### 8️⃣ Propagate Default Route to All Routers

```
EdgeRouter(config)# router ospf 10
EdgeRouter(config-router)# default-information originate
```

> This injects the default route as `O*E2` into the OSPF domain,
> so all routers learn the path to the Internet automatically.

---

### 9️⃣ Auto-Cost Reference Bandwidth (GigabitEthernet=10, FastEthernet=100)

Apply on **ALL routers**:

```
Router(config)# router ospf 10
Router(config-router)# auto-cost reference-bandwidth 10000
```

**Verification:**

| Interface | Bandwidth | Calculation | Cost |
|-----------|-----------|-------------|------|
| GigabitEthernet | 1000 Mbps | 10000 ÷ 1000 | **10** ✅ |
| FastEthernet | 100 Mbps | 10000 ÷ 100 | **100** ✅ |
| Serial (T1) | 1.544 Mbps | 10000 ÷ 1.544 | **6476** |

---

### 🔟 Set OSPF Cost on P2P-1 Serial0/1/1 to 50

```
P2P-1(config)# interface Serial0/1/1
P2P-1(config-if)# ip ospf cost 50
```

> Manual cost overrides the auto-calculated value from reference bandwidth.

---

### 1️⃣1️⃣ Hello & Dead Timers on P2P-1 ↔ BC-1 — Twice the Default

Default timers on a point-to-point link: Hello=10s, Dead=40s.
Twice the default: **Hello=20s, Dead=80s**.

Apply on **both** the P2P-1 and BC-1 interfaces of the connecting link:

```
P2P-1(config)# interface [connecting-interface]
P2P-1(config-if)# ip ospf hello-interval 20
P2P-1(config-if)# ip ospf dead-interval 80
```

```
BC-1(config)# interface [connecting-interface]
BC-1(config-if)# ip ospf hello-interval 20
BC-1(config-if)# ip ospf dead-interval 80
```

> ⚠️ Timers MUST match on both ends of the link or the adjacency will drop!

---

## ✅ Verification Commands

```bash
# Verify OSPF neighbors and states
show ip ospf neighbor

# Verify DR/BDR election on multiaccess network
show ip ospf neighbor detail

# Verify interface costs and timer values
show ip ospf interface brief
show ip ospf interface [interface]

# Verify default route propagation
show ip route ospf

# Verify Router IDs and process ID
show ip ospf

# Verify passive interfaces
show ip protocols
```

---

## 💡 Key Concepts Summary

| Concept | Details |
|---------|---------|
| **Process ID 10** | Locally significant — doesn't need to match between routers |
| **Network statements** | Traditional way to enable OSPF per network with wildcard mask |
| **Interface activation** | Modern way — `ip ospf [pid] area [area]` directly on interface |
| **Router ID** | Manually set for predictability — always best practice |
| **Passive interface** | Advertises network but suppresses Hello packets |
| **OSPF Priority 255** | Guarantees DR role in multiaccess network election |
| **Exit interface default route** | `ip route 0.0.0.0 0.0.0.0 [interface]` |
| **default-information originate** | Floods default route as O*E2 across the OSPF domain |
| **Reference BW 10000** | Makes GigE=10, FastEthernet=100 |
| **Manual cost (`ip ospf cost`)** | Overrides auto-calculated cost on specific interface |
| **Doubled timers** | Hello=20s, Dead=80s — must match on both ends |

---

## 🏅 Score

> 📷 See `score.png` for the full lab score result.

---

## 📝 What I Learned

- How to design and implement a full OSPFv2 network from scratch covering multiple scenarios
- Two different methods to activate OSPF: network statements vs. interface configuration
- How to manually control DR election using OSPF interface priority
- How to inject and propagate a default route to the entire OSPF domain
- How to tune OSPF costs using reference bandwidth and manual cost override
- How Hello/Dead timer customization affects neighbor adjacency — and why both ends must match
- The importance of Router IDs for stability and predictability in OSPF networks
