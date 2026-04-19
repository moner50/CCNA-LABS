# 🌐 Lab — Propagate a Default Route in OSPFv2

## 📌 Lab Info

| Field | Details |
|-------|---------|
| **Course** | CCNA — Routing & Switching |
| **Tool** | Cisco Packet Tracer |
| **Difficulty** | ⭐⭐⭐☆☆ Intermediate |
| **Topic** | OSPFv2 Default Route Propagation to the Internet |

---

## 🎯 Objectives

### Part 1 — Propagate a Default Route
- Configure a static default route on the **edge router** pointing to the Internet
- Use the `default-information originate` command to advertise it via OSPF
- Verify that downstream routers receive the default route

### Part 2 — Verify Connectivity
- Confirm the default route appears in all OSPF routers' routing tables
- Verify that end hosts can successfully reach a web server on the Internet

---

## 🗺️ Topology Overview

> 📷 See `topology.png` for the full diagram.

The topology includes:
- An **edge router** (Gateway) connected to the Internet (simulated ISP)
- **Internal routers** running OSPFv2 in Area 0
- **End hosts** on internal LANs that need Internet access
- A **web server** on the Internet to test connectivity

The edge router is the only one with a path to the Internet.
It must share that path with all other OSPF routers using **default route propagation**.

---

## 🧠 Background Concepts

### What is a Default Route?

A default route (`0.0.0.0/0`) is the **"route of last resort"** — used when no more
specific route exists in the routing table. It essentially says:

> *"If you don't know where to send this packet — send it here."*

In a real network, the edge router (connected to the ISP) holds the default route
and must share it with all internal routers so they can reach the Internet.

---

### How OSPF Propagates a Default Route

By default, OSPF does **NOT** automatically share a default route with neighbors.
You must explicitly tell it to do so using:

```
default-information originate
```

This causes the edge router to generate and flood an **External LSA (Type 5)**
across the OSPF domain, advertising `0.0.0.0/0` to all routers.

---

## ⚙️ Configuration — Part 1

### 🔧 Step 1 — Configure a Static Default Route on the Edge Router

Point the default route to the ISP's IP address (next-hop):

```
EdgeRouter(config)# ip route 0.0.0.0 0.0.0.0 [ISP-next-hop-IP]
```

**Example:**
```
EdgeRouter(config)# ip route 0.0.0.0 0.0.0.0 209.165.200.226
```

> This tells the edge router: *"Send all unknown traffic to the ISP."*

---

### 🔧 Step 2 — Propagate the Default Route via OSPF

Inside the OSPF process, use `default-information originate`:

```
EdgeRouter(config)# router ospf 1
EdgeRouter(config-router)# default-information originate
```

> This injects the default route into the OSPF domain as an **External Type 2 (E2)** route.
> All OSPF neighbors will now receive and install `0.0.0.0/0` in their routing tables.

---

## ✅ Verification — Part 2

### Check the edge router's routing table for the static default route
```
show ip route
```

**Look for:**
```
S*    0.0.0.0/0 [1/0] via 209.165.200.226
```
> `S*` = Static route that is the **gateway of last resort** ✅

---

### Check downstream routers received the OSPF default route
```
show ip route
```

**Look for on internal routers:**
```
O*E2  0.0.0.0/0 [110/1] via 10.0.0.1, 00:01:12, Serial0/0/0
```

| Code | Meaning |
|------|---------|
| `O` | Learned via OSPF |
| `*` | This is the gateway of last resort |
| `E2` | External Type 2 — originated outside OSPF (from the static route) |

---

### Verify the gateway of last resort is set
```
show ip route
```

**Top of output should show:**
```
Gateway of last resort is 10.0.0.1 to network 0.0.0.0
```
> If this line appears on all routers — propagation was successful ✅

---

### Test Internet connectivity from end hosts

On any PC in the topology:
```
ping 209.165.200.254
```
or open the **Web Browser** in Packet Tracer and navigate to the web server's IP.

> All pings and HTTP requests should succeed ✅

---

## 💡 Key Concepts Summary

| Concept | Details |
|---------|---------|
| **Default Route** | `0.0.0.0/0` — the route of last resort for unknown destinations |
| **Static default route** | Configured manually on the edge router pointing to the ISP |
| **`default-information originate`** | Tells OSPF to flood the default route to all neighbors |
| **O\*E2** | OSPF External Type 2 route — how the default route appears on internal routers |
| **Gateway of last resort** | Shown at the top of `show ip route` when a default route exists |
| **Type 5 LSA** | The LSA type used to carry external routes (like the default) across OSPF |

---

## 🔄 Full Configuration Flow

```
[Internet / ISP]
      |
      | (static default route: ip route 0.0.0.0 0.0.0.0 [ISP-IP])
      |
[Edge Router]  ←── default-information originate (floods O*E2 via OSPF)
      |
  [OSPF Area 0]
      |
[Internal Routers]  ←── receive 0.0.0.0/0 as O*E2
      |
[End Hosts]  ←── can now reach the Internet ✅
```

---

## 📝 What I Learned

- How to configure a static default route pointing to an ISP gateway
- Why OSPF does not automatically propagate default routes — and how to enable it
- How `default-information originate` injects the default route as an External LSA
- How to identify a propagated default route in a routing table (`O*E2`)
- How to verify Internet reachability from end hosts using ping and the web browser tool in Packet Tracer
