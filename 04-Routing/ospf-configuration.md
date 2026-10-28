# 🧭 OSPF (Open Shortest Path First) Configuration & Concepts

**OSPF (Open Shortest Path First)** is an open-standard **Link-State routing protocol** defined in RFC 2328 (OSPFv2 for IPv4) and RFC 5340 (OSPFv3 for IPv6). It operates at Layer 3 (IP Protocol `89`) with an **Administrative Distance (AD) of 110** and uses the **Dijkstra SPF (Shortest Path First)** algorithm.

---

## 🔑 Core OSPF Concepts

### 1. Router ID (RID) Selection
Every OSPF router must have a unique 32-bit Router ID. The router selects its RID using this hierarchy:
1. **Manual Configuration:** Highest priority (`router-id X.X.X.X`).
2. **Highest IP on an active Loopback interface:** If no RID is manually configured.
3. **Highest IP on any active physical interface:** If no loopback interface is active.

> 💡 **Best Practice:** Always configure a dedicated `interface Loopback0` or manually set `router-id` so the RID never changes if a physical interface flaps.

### 2. OSPF Neighbor Adjacency States
To exchange routing information, two OSPF neighbors transition through 7 states:
```text
Down ──► Init ──► 2-Way ──► ExStart ──► Exchange ──► Loading ──► Full
```
- **Down:** No OSPF hellos received yet.
- **Init:** Received Hello packet from neighbor, but our own RID is not in the neighbor's received list yet.
- **2-Way:** Bi-directional communication confirmed (our RID is seen). **DR/BDR election occurs here** on broadcast networks.
- **ExStart:** Routers establish master/slave relationship and initial Database Description (DBD) sequence numbers.
- **Exchange:** Routers exchange DBD packets describing their Link-State Database (LSDB).
- **Loading:** Routers send Link-State Requests (LSR) for missing routes and receive Link-State Updates (LSU).
- **Full:** Routers have synchronized, identical LSDBs! Full adjacency established.

### 3. Designated Router (DR) & Backup Designated Router (BDR)
On Multi-Access (Ethernet/Broadcast) networks, routers elect a **DR** and **BDR** to prevent $N(N-1)/2$ full mesh neighbor adjacencies and reduce LSA flooding:
- **DROther** routers form Full adjacency **only** with the DR and BDR.
- **Multicast Addresses Used:**
  - `224.0.0.5`: All OSPF Routers.
  - `224.0.0.6`: All DR and BDR Routers (DROther routers send updates to this address).
- **Election Priority:**
  - Highest OSPF Interface Priority (`ip ospf priority <0-255>`, default is `1`). Priority `0` means "never become DR/BDR".
  - Tie-breaker: Highest Router ID.

### 4. OSPF Metric (Cost Calculation)
$$\text{Cost} = \frac{\text{Reference Bandwidth}}{\text{Interface Bandwidth}}$$
By default, Cisco reference bandwidth is **100 Mbps** ($10^8$ bps):
- FastEthernet (100 Mbps): Cost = $100 / 100 = \mathbf{1}$
- GigabitEthernet (1000 Mbps): Cost = $100 / 1000 = 0.1 \rightarrow \mathbf{1}$ *(Cannot distinguish between FastEthernet and Gigabit!)*
- 10-GigabitEthernet (10 Gbps): Cost = $\mathbf{1}$

> ⚠️ **CCNA Must-Know:** Always adjust reference bandwidth in modern Gigabit networks on **all** OSPF routers:
> ```cisco
> Router(config-router)# auto-cost reference-bandwidth 10000    ! Sets reference to 10 Gbps (10000 Mbps)
> ```

---

## 🛠️ Cisco IOS OSPF Configuration Template

### Method 1: The Modern Interface-Level Method (Recommended)
This approach eliminates wildcard mask calculation errors:

```cisco
enable
configure terminal
hostname R1

! Configure Loopback Interface for stable Router-ID
interface Loopback0
 ip address 1.1.1.1 255.255.255.255
exit

! Enable OSPF Routing Process
router ospf 1
 router-id 1.1.1.1
 auto-cost reference-bandwidth 10000
 ! Stop sending unnecessary Hello packets into local user LANs
 passive-interface GigabitEthernet0/0/0
exit

! Enable OSPF directly under interfaces (Area 0 = Backbone Area)
interface GigabitEthernet0/0/0
 description LAN-Staff
 ip ospf 1 area 0
exit

interface GigabitEthernet0/0/1
 description WAN-Link-to-R2
 ip ospf 1 area 0
 ! Set cost manually if desired
 ip ospf cost 10
exit
```

### Method 2: The Classic Global `network` Statement
```cisco
router ospf 1
 router-id 1.1.1.1
 auto-cost reference-bandwidth 10000
 passive-interface GigabitEthernet0/0/0
 ! syntax: network <network-id> <wildcard-mask> area <area-id>
 network 192.168.10.0 0.0.0.255 area 0
 network 10.0.0.0 0.0.0.3 area 0
exit
```

---

## 🌐 Default Route Propagation in OSPF

To advertise a default Internet route to all other OSPF routers automatically:

```cisco
! On edge router with default route pointing to ISP
Router(config)# ip route 0.0.0.0 0.0.0.0 203.0.113.1
Router(config)# router ospf 1
Router(config-router)# default-information originate
```
*(Use `default-information originate always` to advertise the default route even if an active default route isn't currently installed in the local RIB).*

---

## 🔍 Verification & Troubleshooting Commands

```cisco
! 1. Verify neighbor adjacencies and state (Should show FULL/DR, FULL/BDR, or FULL/DROTHER)
Router# show ip ospf neighbor

! 2. Check OSPF Router-ID, Process ID, and Areas
Router# show ip ospf

! 3. Inspect interface OSPF cost, priority, network type, and timer values
Router# show ip ospf interface GigabitEthernet0/0/1

! 4. Check OSPF routes installed in the routing table (marked with 'O')
Router# show ip route ospf

! 5. View Link-State Database (LSDB) summary
Router# show ip ospf database

! 6. Clear OSPF process (forces new Hello exchange and DR election)
Router# clear ip ospf process
Reset ALL OSPF processes? [no]: yes
```

---

## ⚠️ Common OSPF Adjacency Troubleshooting Checkpoints

If two OSPF routers are stuck in `Down`, `Init`, or `ExStart`, check these common mismatches:
1. **Subnet Mask Mismatch:** Both interfaces on a point-to-point link must belong to the exact same subnet.
2. **Hello / Dead Timer Mismatch:** Hello (default 10s on broadcast) and Dead (default 40s) timers must match.
3. **Area ID Mismatch:** Both interfaces must belong to the same Area (e.g., Area 0).
4. **Area Type Flags:** Normal vs. Stub / NSSA must match.
5. **MTU Mismatch:** Causes adjacency to get stuck in **`ExStart/Exchange`**!
6. **Passive Interface:** If `passive-interface` is configured on the link connecting two routers, Hello packets are silenced.
7. **Duplicate Router ID:** Both routers share the exact same RID.
