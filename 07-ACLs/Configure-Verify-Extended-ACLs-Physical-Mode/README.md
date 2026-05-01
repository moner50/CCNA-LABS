# 🛡️ Lab — Configure and Verify Extended IPv4 ACLs (Physical Mode)

## 📌 Lab Info

| Field | Details |
|-------|---------|
| **Course** | CCNA — Network Security |
| **Tool** | Cisco Packet Tracer — Physical Mode |
| **Difficulty** | ⭐⭐⭐⭐⭐ Advanced |
| **Score** | 🏅 97% |
| **Topic** | VLANs + Trunking + Inter-VLAN Routing + SSH + Extended ACLs |

> 📥 **Download the Packet Tracer file** and test the full configuration yourself.

---

## 🎯 Objectives

- Configure VLANs and assign ports
- Configure trunk interfaces between switches and router
- Configure Inter-VLAN Routing (Router-on-a-Stick)
- Secure device access using SSH
- Implement Extended ACLs to enforce security policies
- Verify all configurations using show commands and connectivity tests

---

## 🗺️ Topology Overview

> 📷 See `topology.png` for the full network diagram.
> 📷 See `score.png` for the lab score — **97%** ✅

---

## 📋 VLAN Table

| VLAN | Name | Interface Assigned |
|------|------|--------------------|
| 20 | Management | S2: F0/5 |
| 30 | Operations | S1: F0/6 |
| 40 | Sales | S2: F0/18 |
| 999 | ParkingLot | S1: F0/2-4, F0/7-24, G0/1-2 / S2: F0/2-4, F0/6-17, F0/19-24, G0/1-2 |
| 1000 | Native | N/A |

---

## 📋 Addressing Table

| Device | Interface | IP Address |
|--------|-----------|------------|
| R1 | G0/0/0 | 172.16.1.1/24 |
| R1 | G0/0/1.20 | 10.20.0.1/24 |
| R1 | G0/0/1.30 | 10.30.0.1/24 |
| R1 | G0/0/1.40 | 10.40.0.1/24 |
| R1 | G0/0/1.999 | N/A |
| R1 | G0/0/1.1000 | N/A |
| Server-A | NIC | 172.16.1.2/24 |

---

## ⚙️ Full Configuration

---

### Part 1 — VLAN Configuration on S1 and S2

```
Switch(config)# vlan 20
Switch(config-vlan)# name Management
Switch(config)# vlan 30
Switch(config-vlan)# name Operations
Switch(config)# vlan 40
Switch(config-vlan)# name Sales
Switch(config)# vlan 999
Switch(config-vlan)# name ParkingLot
Switch(config)# vlan 1000
Switch(config-vlan)# name Native
```

**Assign ports to VLANs and shutdown ParkingLot:**
```
Switch(config)# interface range f0/2-4, f0/7-24, g0/1-2
Switch(config-if-range)# switchport mode access
Switch(config-if-range)# switchport access vlan 999
Switch(config-if-range)# shutdown
```

---

### Part 2 — Trunk Configuration (S1 and S2)

**F0/1 on both switches:**
```
Switch(config)# interface f0/1
Switch(config-if)# switchport mode trunk
Switch(config-if)# switchport trunk native vlan 1000
Switch(config-if)# switchport trunk allowed vlan 20,30,40,999,1000
Switch(config-if)# exit
```

**F0/5 on S1 only (toward R1):**
```
S1(config)# interface f0/5
S1(config-if)# switchport mode trunk
S1(config-if)# switchport trunk native vlan 1000
S1(config-if)# switchport trunk allowed vlan 20,30,40,999,1000
S1(config-if)# end
S1# copy running-config startup-config
```

---

### Part 3 — Inter-VLAN Routing on R1 (Router-on-a-Stick)

```
R1(config)# interface g0/0/1
R1(config-if)# no shutdown
R1(config-if)# exit

R1(config)# interface g0/0/1.20
R1(config-subif)# description Management Network
R1(config-subif)# encapsulation dot1Q 20
R1(config-subif)# ip address 10.20.0.1 255.255.255.0
R1(config-subif)# exit

R1(config)# interface g0/0/1.30
R1(config-subif)# description Operations Network
R1(config-subif)# encapsulation dot1Q 30
R1(config-subif)# ip address 10.30.0.1 255.255.255.0
R1(config-subif)# exit

R1(config)# interface g0/0/1.40
R1(config-subif)# description Sales Network
R1(config-subif)# encapsulation dot1Q 40
R1(config-subif)# ip address 10.40.0.1 255.255.255.0
R1(config-subif)# exit

R1(config)# interface g0/0/1.999
R1(config-subif)# description ParkingLot
R1(config-subif)# encapsulation dot1Q 999
R1(config-subif)# exit

R1(config)# interface g0/0/1.1000
R1(config-subif)# description Native VLAN
R1(config-subif)# encapsulation dot1Q 1000 native
R1(config-subif)# exit
```

---

### Part 4 — SSH Configuration (R1, S1, S2)

Apply on **each device**:
```
Device(config)# username SSHadmin secret $cisco123!
Device(config)# ip domain-name ccna-lab.com
Device(config)# crypto key generate rsa
How many bits in the modulus [512]: 1024
Device(config)# line vty 0 4
Device(config-line)# transport input ssh
Device(config-line)# login local
Device(config-line)# exit
```

---

### Part 5 — Extended ACL Security Policies

#### 🔒 Security Policies Summary

| Policy | Source | Destination | Protocol | Action |
|--------|--------|-------------|----------|--------|
| 1 | Sales `10.40.0.0/24` | Management `10.20.0.0/24` | SSH (22) | ❌ Deny |
| 2 | Sales `10.40.0.0/24` | Server-A `172.16.1.2` | HTTP/HTTPS | ❌ Deny |
| 3 | Sales `10.40.0.0/24` | Operations + Management | ICMP echo | ❌ Deny |
| 4 | Operations `10.30.0.0/24` | Sales `10.40.0.0/24` | ICMP echo | ❌ Deny |
| — | Any | Any | All other | ✅ Permit |

---

#### SALES_POLICY ACL

```
R1(config)# ip access-list extended SALES_POLICY
R1(config-ext-nacl)# remark Policy1-Block SSH to Management
R1(config-ext-nacl)# deny tcp 10.40.0.0 0.0.0.255 10.20.0.0 0.0.0.255 eq 22
R1(config-ext-nacl)# remark Policy2-Block HTTP and HTTPS to Server-A
R1(config-ext-nacl)# deny tcp 10.40.0.0 0.0.0.255 host 172.16.1.2 eq 80
R1(config-ext-nacl)# deny tcp 10.40.0.0 0.0.0.255 host 172.16.1.2 eq 443
R1(config-ext-nacl)# remark Policy3-Block ICMP to Operations and Management
R1(config-ext-nacl)# deny icmp 10.40.0.0 0.0.0.255 10.30.0.0 0.0.0.255 echo
R1(config-ext-nacl)# deny icmp 10.40.0.0 0.0.0.255 10.20.0.0 0.0.0.255 echo
R1(config-ext-nacl)# permit ip any any
R1(config-ext-nacl)# exit

R1(config)# interface g0/0/1.40
R1(config-subif)# ip access-group SALES_POLICY in
```

#### OPS_POLICY ACL

```
R1(config)# ip access-list extended OPS_POLICY
R1(config-ext-nacl)# remark Policy4-Block ICMP to Sales
R1(config-ext-nacl)# deny icmp 10.30.0.0 0.0.0.255 10.40.0.0 0.0.0.255 echo
R1(config-ext-nacl)# permit ip any any
R1(config-ext-nacl)# exit

R1(config)# interface g0/0/1.30
R1(config-subif)# ip access-group OPS_POLICY in
```

---

## 🧪 Verification Tests

| From | To | Protocol | Expected |
|------|----|----------|----------|
| Sales `10.40.0.x` | Management `10.20.0.x` | SSH | ❌ Blocked |
| Sales `10.40.0.x` | Server-A `172.16.1.2` | HTTP | ❌ Blocked |
| Sales `10.40.0.x` | Server-A `172.16.1.2` | HTTPS | ❌ Blocked |
| Sales `10.40.0.x` | Operations `10.30.0.x` | ICMP echo | ❌ Blocked |
| Sales `10.40.0.x` | Management `10.20.0.x` | ICMP echo | ❌ Blocked |
| Sales `10.40.0.x` | Server-A `172.16.1.2` | ICMP echo | ✅ Allowed |
| Operations `10.30.0.x` | Sales `10.40.0.x` | ICMP echo | ❌ Blocked |
| Operations `10.30.0.x` | Management `10.20.0.x` | ICMP echo | ✅ Allowed |
| Management `10.20.0.x` | Sales `10.40.0.x` | SSH | ✅ Allowed |

---

## 🔍 Verification Commands

```bash
show vlan brief                        # Verify VLANs and port assignments
show interfaces trunk                  # Verify trunk interfaces
show ip interface brief                # Verify subinterface IPs
show ip route                          # Verify Inter-VLAN routes
show ip ssh                            # Verify SSH is enabled
show access-lists                      # Verify ACLs and match counters
show ip interface g0/0/1.40            # Verify SALES_POLICY applied
show ip interface g0/0/1.30            # Verify OPS_POLICY applied
```

---

## 💡 Key Concepts Summary

| Concept | Details |
|---------|---------|
| **Router-on-a-Stick** | One physical interface with multiple subinterfaces — one per VLAN |
| **Native VLAN 1000** | Must match on both ends of every trunk link |
| **ParkingLot VLAN 999** | All unused ports → assigned + shutdown for security |
| **SSH over Telnet** | `transport input ssh` — blocks Telnet completely |
| **Extended ACL placement** | Inbound on the source VLAN subinterface — closest to source |
| **`echo` keyword** | Matches only ICMP echo requests (pings) — not echo replies |
| **`remark`** | Documents each ACL section — best practice for readability |

---

## 📝 What I Learned

- How to build a complete enterprise-like network from scratch in Physical Mode
- How Router-on-a-Stick enables inter-VLAN routing with a single physical link
- Why native VLAN must match on both ends of a trunk — mismatches cause issues
- How SSH secures remote access — and why Telnet should always be disabled
- How to translate business security policies into precise ACL statements
- Why `echo` keyword is important — it only blocks outgoing pings, not return replies
- The satisfaction of seeing 97% on a complex multi-part lab 🏅
