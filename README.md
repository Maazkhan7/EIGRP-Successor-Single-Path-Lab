<div align="center">

# 🌐 EIGRP Successor – Single Path Lab

### Understanding DUAL, Successors & Feasible Distance in Cisco EIGRP (AS 1)

![Cisco](https://img.shields.io/badge/Cisco-Packet%20Tracer-1BA0D7?style=for-the-badge&logo=cisco&logoColor=white)
![Protocol](https://img.shields.io/badge/Protocol-EIGRP-blue?style=for-the-badge)
![AS](https://img.shields.io/badge/AS-1-orange?style=for-the-badge)
![Routers](https://img.shields.io/badge/Routers-2911-green?style=for-the-badge)
![Level](https://img.shields.io/badge/Level-CCNA%20%2F%20CCNP-red?style=for-the-badge)
![Status](https://img.shields.io/badge/Status-Completed-success?style=for-the-badge)

</div>

---

## 📌 Project Overview

This lab demonstrates the **EIGRP Successor** concept in the simplest possible scenario: **one single path** to every remote network.

Two Cisco 2911 routers (**R1** and **R2**) are connected through a serial WAN link. Each router has its own LAN. With EIGRP AS 1 running on both, every remote network is reachable through **exactly one next hop** — so the EIGRP topology table shows **`1 successors`** for each route and **no Feasible Successor** (no backup path exists).

This is the foundation lab for understanding how **DUAL (Diffusing Update Algorithm)** selects the best loop-free path, and why a second path (Feasible Successor) matters for fast convergence.

---

## 🎯 Objectives

- ✅ Configure EIGRP AS 1 on two routers over a serial link
- ✅ Form an EIGRP neighbor adjacency and verify it
- ✅ Read and interpret the **EIGRP topology table**
- ✅ Identify the **Successor**, **Feasible Distance (FD)** and **Reported Distance (RD)**
- ✅ Confirm the route is installed in the routing table with **AD 90**
- ✅ Prove the forwarding path with **traceroute**
- ✅ Understand what happens when only a single path exists (no Feasible Successor)

---

## 🗺️ Network Topology

```
   192.168.1.0/24                                              192.168.2.0/24
 ┌───────────────┐                                           ┌───────────────┐
 │               │                                           │               │
PC1 ─┐                                                              ┌─ PC4
PC2 ─┼─ Switch0 ── Gi0/0 ┌──────┐ Se0/0/0 ═════ Se0/0/0 ┌──────┐ Gi0/0 ── Switch1 ─┼─ PC5
PC3 ─┘   (2950)   .1     │  R1  │ .1    1.1.1.0/24   .2 │  R2  │  .1    (2950)      └─ PC6
                        │ 2911 │                        │ 2911 │
                        └──────┘                        └──────┘
                     Router ID: 192.168.1.1          Router ID: 192.168.2.1
                         EIGRP AS 1                      EIGRP AS 1
```

> 📷 Add your Packet Tracer screenshot here: `![Topology](images/topology.png)`

---

## 📋 IP Addressing Table

| Device | Interface | IP Address | Subnet Mask | Connected To |
|:------:|:---------:|:----------:|:-----------:|:------------:|
| **R1** | Gi0/0 | 192.168.1.1 | 255.255.255.0 | Switch0 (LAN 1) |
| **R1** | Se0/0/0 | 1.1.1.1 | 255.255.255.0 | R2 Se0/0/0 |
| **R2** | Gi0/0 | 192.168.2.1 | 255.255.255.0 | Switch1 (LAN 2) |
| **R2** | Se0/0/0 | 1.1.1.2 | 255.255.255.0 | R1 Se0/0/0 |
| PC1 | NIC | 192.168.1.2 | 255.255.255.0 | Switch0 |
| PC2 | NIC | 192.168.1.3 | 255.255.255.0 | Switch0 |
| PC3 | NIC | 192.168.1.4 | 255.255.255.0 | Switch0 |
| PC4 | NIC | 192.168.2.2 | 255.255.255.0 | Switch1 |
| PC5 | NIC | 192.168.2.3 | 255.255.255.0 | Switch1 |
| PC6 | NIC | 192.168.2.4 | 255.255.255.0 | Switch1 |

> Default gateway for LAN 1 PCs: `192.168.1.1` · Default gateway for LAN 2 PCs: `192.168.2.1`

---

## ⚙️ Configuration

> Standard EIGRP configuration for this topology. Adjust to match your own running-config if it differs.

### 🔹 R1

```bash
enable
configure terminal
hostname R1

interface GigabitEthernet0/0
 ip address 192.168.1.1 255.255.255.0
 no shutdown

interface Serial0/0/0
 ip address 1.1.1.1 255.255.255.0
 no shutdown

router eigrp 1
 network 1.1.1.0 0.0.0.255
 network 192.168.1.0 0.0.0.255
 no auto-summary

end
write memory
```

### 🔹 R2

```bash
enable
configure terminal
hostname R2

interface GigabitEthernet0/0
 ip address 192.168.2.1 255.255.255.0
 no shutdown

interface Serial0/0/0
 ip address 1.1.1.2 255.255.255.0
 no shutdown

router eigrp 1
 network 1.1.1.0 0.0.0.255
 network 192.168.2.0 0.0.0.255
 no auto-summary

end
write memory
```

---

## ✅ Verification

### 1️⃣ EIGRP Neighbor Table

**R1 → `show ip eigrp neighbors`**

| H | Address | Interface | Hold (sec) | Uptime | SRTT (ms) | RTO | Q Cnt | Seq Num |
|:-:|:-------:|:---------:|:----------:|:------:|:---------:|:---:|:-----:|:-------:|
| 0 | 1.1.1.2 | Se0/0/0 | 14 | 00:04:49 | 40 | 1000 | 0 | 4 |

**R2 → `show ip eigrp neighbors`**

| H | Address | Interface | Hold (sec) | Uptime | SRTT (ms) | RTO | Q Cnt | Seq Num |
|:-:|:-------:|:---------:|:----------:|:------:|:---------:|:---:|:-----:|:-------:|
| 0 | 1.1.1.1 | Se0/0/0 | 13 | 00:01:25 | 40 | 1000 | 0 | 3 |

📝 **Each router sees exactly one neighbor** → one path → one successor.

---

### 2️⃣ EIGRP Topology Table (⭐ Main Focus)

**R1 → `show ip eigrp topology`**

```
IP-EIGRP Topology Table for AS 1/ID(192.168.1.1)

P 1.1.1.0/24, 1 successors, FD is 2169856
        via Connected, Serial0/0/0
P 192.168.1.0/24, 1 successors, FD is 28160
        via Connected, GigabitEthernet0/0
P 192.168.2.0/24, 1 successors, FD is 2172416
        via 1.1.1.2 (2172416/28160), Serial0/0/0
```

**R2 → `show ip eigrp topology`**

```
IP-EIGRP Topology Table for AS 1/ID(192.168.2.1)

P 1.1.1.0/24, 1 successors, FD is 2169856
        via Connected, Serial0/0/0
P 192.168.1.0/24, 1 successors, FD is 2172416
        via 1.1.1.1 (2172416/28160), Serial0/0/0
P 192.168.2.0/24, 1 successors, FD is 28160
        via Connected, GigabitEthernet0/0
```

#### 🔍 How to read it

| Field | Meaning | Value in this lab |
|:------|:--------|:------------------|
| **P** | Passive state — route is stable, no recomputation running | All routes are **P** ✅ |
| **1 successors** | Number of best (lowest-metric) next hops | **1** → single path |
| **FD** | Feasible Distance — total best metric from this router to the destination | 2172416 (remote LAN) |
| **(2172416/28160)** | **(FD / RD)** — RD is the metric *the neighbor* reports | RD = 28160 (neighbor's LAN) |
| **via 1.1.1.x** | Next hop = the Successor | R1→1.1.1.2 · R2→1.1.1.1 |
| **via Connected** | Directly connected network | Local interfaces |

#### 🧮 Why is FD = 2172416?

EIGRP metric = `256 × ( 10⁷ / lowest bandwidth (Kbps) + total delay (tens of µs) )`

```
Lowest bandwidth on path  : Serial link  = 1544 Kbps  → 10,000,000 / 1544 = 6476
Total delay on path       : Serial (2000) + remote Gi0/0 (10) = 2010

Metric = 256 × (6476 + 2010) = 256 × 8486 = 2,172,416 ✅
```

The **RD = 28160** is the neighbor's own cost to its connected LAN (`256 × (100 + 10)`), which is smaller than the FD — the first requirement of the **Feasibility Condition**.

---

### 3️⃣ Routing Table

**R1 → `show ip route`**

```
     1.0.0.0/24 is subnetted
C       1.1.1.0/24 is directly connected, Serial0/0/0
L       1.1.1.1/32 is directly connected, Serial0/0/0
     192.168.1.0/24 is variably subnetted, 2 subnets, 2 masks
C       192.168.1.0/24 is directly connected, GigabitEthernet0/0
L       192.168.1.1/32 is directly connected, GigabitEthernet0/0
D    192.168.2.0/24 [90/2172416] via 1.1.1.2, 00:04:47, Serial0/0/0
```

**R2 → `show ip route`**

```
D    192.168.1.0/24 [90/2172416] via 1.1.1.1, 00:01:39, Serial0/0/0
C    192.168.2.0/24 is directly connected, GigabitEthernet0/0
L    192.168.2.1/32 is directly connected, GigabitEthernet0/0
```

| Code | Meaning |
|:----:|:--------|
| **D** | Learned via EIGRP (internal) |
| **90** | Administrative Distance of internal EIGRP |
| **2172416** | EIGRP composite metric — **same as the FD** in the topology table |

📌 The route in the routing table **always matches the Successor** in the topology table.

---

### 4️⃣ Traceroute (Path Proof)

**R1 → `traceroute 192.168.2.1`**
```
  1   1.1.1.2     0 msec    0 msec    1 msec
```

**R2 → `traceroute 192.168.1.1`**
```
  1   1.1.1.1     8 msec    1 msec    0 msec
```

✅ Each traceroute shows a single hop through the serial link — the only path available. Repeated runs confirm consistent forwarding through the same successor every time.

---

## 🧠 Key Concepts Explained

### 🔹 Successor
The **next-hop router** with the **lowest total metric** (best path) to a destination. It is placed in the routing table.

### 🔹 Feasible Distance (FD)
The **lowest calculated metric** from the local router to the destination network (including the cost to reach the neighbor).

### 🔹 Reported Distance (RD) / Advertised Distance (AD)
The metric a **neighbor advertises** for the same destination — i.e. the neighbor's own cost to reach it.

### 🔹 Feasible Successor (FS)
A **backup next hop** whose **RD < current FD** (Feasibility Condition). It lets EIGRP switch instantly without querying.

### 🔹 DUAL
The **Diffusing Update Algorithm** — EIGRP's engine that guarantees **loop-free** best-path selection.

### 🔹 Passive vs Active
- **Passive (P)** → route is stable and usable
- **Active (A)** → router lost its successor with no feasible successor and is **querying neighbors**

---

## 🔄 What Happens If the Link Fails?

| Scenario | Single Path (this lab) | Dual Path (with Feasible Successor) |
|:---------|:----------------------:|:-----------------------------------:|
| Successors | 1 | 1 |
| Feasible Successor | ❌ None | ✅ Present |
| On primary link failure | Route goes **Active** → router sends **Queries** | Backup promoted **instantly** |
| Convergence | Slower (query/reply process) | Near-instant |
| Redundancy | None | Yes |

> 💡 This is exactly why production networks use **redundant links** — so DUAL always has a Feasible Successor ready.

---

## 🎓 Key Takeaways

1. 🔸 `1 successors` in the topology table = **one best path** to the destination
2. 🔸 The **successor is the next hop** installed in the routing table
3. 🔸 **FD in the topology table = metric in the routing table** (`[90/2172416]`)
4. 🔸 **RD (28160) < FD (2172416)** → feasibility condition met by the neighbor's advertisement
5. 🔸 With a **single path there is no Feasible Successor** → no instant backup
6. 🔸 Internal EIGRP routes carry **AD = 90**
7. 🔸 Neighbor adjacency (Hold, Uptime, SRTT, RTO, Q Cnt = 0) shows a healthy, stable relationship
8. 🔸 Directly connected networks show `via Connected` with a successor of their own

---

## 🛠️ Useful Commands

```bash
show ip eigrp neighbors            # Verify adjacency
show ip eigrp topology             # Successors, FD, RD (passive routes)
show ip eigrp topology all-links   # Shows non-successor/FS paths too
show ip route eigrp                # EIGRP routes in the routing table
show ip protocols                  # AS number, networks, AD, router ID
show ip eigrp interfaces           # Interfaces participating in EIGRP
traceroute 192.168.2.1             # Verify forwarding path
```

---

## 🧪 Test It Yourself

1. Build the topology in Cisco Packet Tracer (2× 2911, 2× 2950-24, 6× PC)
2. Apply the configurations above
3. Verify neighbors → topology → routing table → traceroute
4. **Challenge:** Add a **second link** between R1 and R2 and watch the topology table — can you create a **Feasible Successor**?
5. **Challenge:** Shut down `Serial0/0/0` and observe the route transition to **Active** state

---

## 📁 Repository Structure

```
📦 EIGRP-Successor-Single-Path
 ┣ 📜 README.md
 ┣ 📂 images
 ┃ ┣ 🖼️ topology.png
 ┃ ┣ 🖼️ r1-topology-neighbors-route.png
 ┃ ┣ 🖼️ r1-traceroute.png
 ┃ ┣ 🖼️ r2-topology-neighbors-route.png
 ┃ ┗ 🖼️ r2-traceroute.png
 ┗ 📂 lab-file
   ┗ 📦 EIGRP-Successor-Single-Path.pkt
```

---

## 🔗 Related Topics

`EIGRP` · `DUAL` · `Successor` · `Feasible Successor` · `Feasible Distance` · `Reported Distance` · `Topology Table` · `Administrative Distance` · `Route Convergence`

---

## 👨‍💻 Author

**Maaz Khan**
CCNA-Certified Network & NOC Engineer

[![LinkedIn](https://img.shields.io/badge/LinkedIn-maazkhanms-0A66C2?style=for-the-badge&logo=linkedin&logoColor=white)](https://www.linkedin.com/in/maazkhanms)
[![GitHub](https://img.shields.io/badge/GitHub-maazkhanms-181717?style=for-the-badge&logo=github&logoColor=white)](https://github.com/maazkhanms)

---

<div align="center">

⭐ **If you found this lab helpful, please star the repository!** ⭐

*Part of my ongoing Cisco Packet Tracer lab series — routing, switching, security & more.*

</div>
