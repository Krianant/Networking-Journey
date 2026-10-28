# 📖 Cisco IOS Routing Table Analysis

The **Routing Table** (RIB - Routing Information Base) is the brain of a router. Whenever a packet arrives at an interface, the router inspects the packet's destination IP address and compares it against entries in the routing table to decide the outgoing egress interface and next-hop.

---

## 🏷️ Anatomy of `show ip route`

When you execute `show ip route` on a Cisco router, the output is structured into two main sections:
1. **Codes Legend:** Key defining how each route was learned.
2. **Routing Table Entries:** List of reachable destination networks.

```cisco
Router# show ip route
Codes: L - local, C - connected, S - static, R - RIP, M - mobile, B - BGP
       D - EIGRP, EX - EIGRP external, O - OSPF, IA - OSPF inter area 
       N1 - OSPF NSSA external type 1, N2 - OSPF NSSA external type 2
       E1 - OSPF external type 1, E2 - OSPF external type 2
       i - IS-IS, su - IS-IS summary, L1 - IS-IS level-1, L2 - IS-IS level-2
       ia - IS-IS inter area, * - candidate default, U - per-user static route
       o - ODR, P - periodic downloaded static route, H - NHRP, l - LISP
       a - application route
       + - replicated route, % - next hop override, p - overrides from PfR

Gateway of last resort is 10.0.0.2 to network 0.0.0.0

      10.0.0.0/8 is variably subnetted, 2 subnets, 2 masks
C        10.0.0.0/30 is directly connected, GigabitEthernet0/0/1
L        10.0.0.1/32 is directly connected, GigabitEthernet0/0/1
S     192.168.20.0/24 [1/0] via 10.0.0.2
O     192.168.30.0/24 [110/2] via 10.0.0.2, 00:04:15, GigabitEthernet0/0/1
S*    0.0.0.0/0 [1/0] via 10.0.0.2
```

---

## 🔍 Breaking Down a Single Route Entry

Let's dissect this OSPF route entry:
```text
  O     192.168.30.0/24 [110/2] via 10.0.0.2, 00:04:15, GigabitEthernet0/0/1
  │            │          │  │         │         │               │
  │            │          │  │         │         │               └─ Outgoing Egress Interface
  │            │          │  │         │         └─ Route Timestamp (Uptime)
  │            │          │  │         └─ Next-Hop IP Address
  │            │          │  └─ Metric / Cost (2)
  │            │          └─ Administrative Distance (AD = 110)
  │            └─ Destination Network with CIDR prefix
  └─ Route Source Code (O = OSPF)
```

### Route Codes Breakdown:
- **`C` (Connected):** Network directly attached to an active router interface configured with an IP address and in `up/up` state.
- **`L` (Local):** A `/32` host route representing the specific IP address assigned directly to the router's own interface (introduced in Cisco IOS 15.0).
- **`S` (Static):** Manually configured static route.
- **`S*` (Candidate Default):** Static default route (`0.0.0.0/0`).
- **`O` (OSPF):** Learned via OSPF dynamic routing protocol.
- **`D` (EIGRP):** Learned via EIGRP.
- **`B` (BGP):** Learned via Border Gateway Protocol.

---

## ⚖️ Administrative Distance (AD)

**Administrative Distance** is the measure of trustworthiness (believability) of a routing information source. Lower numbers are preferred!

| Route Source | Administrative Distance (AD) |
| :--- | :---: |
| **Directly Connected (`C`)** | **0** |
| **Static Route (`S`)** | **1** |
| **EIGRP Summary Route** | **5** |
| **eBGP (External BGP)** | **20** |
| **EIGRP Internal (`D`)** | **90** |
| **OSPF (`O`)** | **110** |
| **IS-IS** | **115** |
| **RIP** | **120** |
| **EIGRP External** | **170** |
| **iBGP (Internal BGP)** | **200** |
| **Unreachable / Unknown** | **255** (Dropped) |

---

## 🎯 The Three Golden Rules of Packet Forwarding

When a router receives a packet with destination IP `192.168.1.155`:

### Rule 1: Longest Prefix Match (LPM) Trumps Everything!
The router selects the route entry with the **most specific prefix** (the longest subnet mask / most binary `1`s), regardless of the routing protocol or Administrative Distance!

*Example:*
- Route A: `192.168.1.0/24` (Static Route, AD=1)
- Route B: `192.168.1.128/25` (OSPF Route, AD=110)
- Route C: `0.0.0.0/0` (Default Route, AD=1)

*Decision:* `192.168.1.155` matches both Route A and Route B, but **Route B (/25)** is selected because `/25` has 25 network bits vs 24 bits for `/24`!

### Rule 2: Administrative Distance breaks ties between identical prefixes
If two different routing protocols advertise the exact same destination prefix (e.g. `10.1.1.0/24` via OSPF and EIGRP), the router installs the one with the lowest AD (EIGRP AD=90 wins over OSPF AD=110).

### Rule 3: Metric breaks ties within the same routing protocol
If two paths to the exact same prefix are learned via the same protocol (e.g. OSPF), the path with the lowest metric/cost is selected. If metrics are equal, **Equal-Cost Multi-Path (ECMP)** load balancing occurs.
