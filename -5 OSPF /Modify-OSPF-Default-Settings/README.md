# ⚙️ Lab — Modify OSPF Default Settings

## 📌 Lab Info

| Field | Details |
|-------|---------|
| **Course** | CCNA — Routing & Switching |
| **Tool** | Cisco Packet Tracer |
| **Difficulty** | ⭐⭐⭐☆☆ Intermediate |
| **Topic** | OSPF Hello/Dead Timers & Bandwidth Adjustment |

---

## 🎯 Objectives

### Part 1 — Modify OSPF Default Settings
- Change the **Hello timer** on OSPF interfaces
- Change the **Dead timer** on OSPF interfaces
- Adjust the **bandwidth** of a link to influence OSPF cost

### Part 2 — Verify Connectivity
- Confirm OSPF neighbor adjacencies are re-established after timer changes
- Verify all end devices have full connectivity
- Check that OSPF cost reflects the new bandwidth setting

---

## 🗺️ Topology Overview

> 📷 See `topology.png` for the full diagram.

OSPF is **pre-configured** in this lab. The routers already have full adjacency and all
end devices have connectivity at the start. Your task is to modify OSPF defaults
without breaking — and then restore — full connectivity.

---

## 🧠 Background Concepts

### 🕐 Hello & Dead Timers

OSPF uses two timers to maintain neighbor relationships:

| Timer | Default | Purpose |
|-------|---------|---------|
| **Hello Timer** | 10 sec (broadcast) / 30 sec (serial) | How often Hello packets are sent |
| **Dead Timer** | 40 sec (broadcast) / 120 sec (serial) | How long to wait before declaring a neighbor down |

> ⚠️ **Critical Rule:** Hello and Dead timers **must match** on both sides of a link.
> If they don't match, OSPF neighbors will **NOT** form adjacency.

---

### 💰 OSPF Cost & Bandwidth

OSPF uses **cost** as its metric to choose the best path.

```
Cost = Reference Bandwidth / Interface Bandwidth
     = 100 Mbps / Interface Bandwidth (default)
```

| Interface | Bandwidth | Default Cost |
|-----------|-----------|--------------|
| Serial (T1) | 1.544 Mbps | 64 |
| FastEthernet | 100 Mbps | 1 |
| GigabitEthernet | 1000 Mbps | 1 |

> If two different-speed interfaces have the same cost (e.g., Fa and Gi both = 1),
> OSPF cannot distinguish between them. Fix this by adjusting the reference bandwidth
> or setting the cost manually.

---

## ⚙️ Configuration — Part 1

### 🔧 Step 1 — Change Hello and Dead Timers

Apply on the **interface level** of each router (must match on both ends):

```
Router(config)# interface [interface]
Router(config-if)# ip ospf hello-interval [seconds]
Router(config-if)# ip ospf dead-interval [seconds]
```

**Example — setting Hello to 5 sec and Dead to 20 sec:**
```
Router(config)# interface GigabitEthernet0/0
Router(config-if)# ip ospf hello-interval 5
Router(config-if)# ip ospf dead-interval 20
```

> 💡 A common rule: Dead interval = 4 × Hello interval

> ⚠️ After changing timers, OSPF adjacency will **drop** temporarily until
> both sides are updated. This is expected behavior.

---

### 🔧 Step 2 — Adjust Bandwidth of a Link

Set the **bandwidth** value on the interface to reflect the true link speed,
which directly affects the OSPF cost:

```
Router(config)# interface [interface]
Router(config-if)# bandwidth [kilobits-per-second]
```

**Example — set a serial link to 512 Kbps:**
```
Router(config)# interface Serial0/0/0
Router(config-if)# bandwidth 512
```

> ⚠️ The `bandwidth` command only affects **routing metrics** — it does NOT
> change the actual speed of the interface.

**Alternatively — set cost directly:**
```
Router(config-if)# ip ospf cost [value]
```

---

## ✅ Verification — Part 2

### Check Hello and Dead timer values on an interface
```
show ip ospf interface [interface]
```

**Expected output (look for timer values):**
```
Timer intervals configured, Hello 5, Dead 20, Wait 20, Retransmit 5
```

---

### Check OSPF neighbor adjacency is restored
```
show ip ospf neighbor
```

**Expected output:**
```
Neighbor ID     Pri   State           Dead Time   Address         Interface
2.2.2.2           1   FULL/DR         00:00:18    192.168.1.2     GigE0/0
```

> State must be **FULL** for connectivity to work ✅

---

### Check OSPF cost reflects the new bandwidth
```
show ip ospf interface [interface]
```

**Look for:**
```
Process ID 1, Router ID 1.1.1.1, Network Type BROADCAST, Cost: 195
```

---

### Verify full end-to-end connectivity
```
ping [destination-ip]
```
> Test from end devices — all pings should succeed ✅

---

## 💡 Key Concepts Summary

| Concept | Details |
|---------|---------|
| **Hello Timer** | Must match on both ends of a link |
| **Dead Timer** | Must match on both ends — usually 4× Hello |
| **Mismatched timers** | Neighbors won't form adjacency (stuck at INIT/2WAY) |
| **Bandwidth command** | Changes OSPF cost calculation, not actual link speed |
| **OSPF Cost formula** | 100 Mbps ÷ Interface Bandwidth |
| **Manual cost** | `ip ospf cost` overrides the calculated cost |

---

## 📝 What I Learned

- How to fine-tune OSPF behavior by modifying Hello and Dead timer intervals
- Why **both sides of a link must have matching timers** — and what happens if they don't
- How OSPF calculates cost from interface bandwidth
- How to use the `bandwidth` command to influence route selection
- How to verify OSPF timer and cost settings using `show ip ospf interface`
- That timer changes temporarily drop adjacency — this is normal and expected
