# 🧭 Static Routing in Cisco IOS

A **static route** is a manually configured routing entry in a router's routing table that defines an explicit path for packets destined to a specific remote network.

---

## ⚖️ Static vs. Dynamic Routing

| Feature | Static Routing | Dynamic Routing (OSPF, EIGRP, BGP) |
| :--- | :--- | :--- |
| **Configuration** | Manually entered by administrator | Automatically discovered via protocol messages |
| **CPU / Bandwidth Overhead** | Zero CPU overhead, zero network bandwidth | Consumes router CPU, RAM, and link bandwidth |
| **Scalability** | Best for small networks (stub networks) | Scales easily to hundreds or thousands of routers |
| **Topology Changes** | Cannot adapt automatically; requires manual admin intervention | Automatically detects link failures and recalculates path |
| **Security** | Highly secure (no routing advertisements sent) | Requires authentication to prevent rogue route injection |

---

## 📝 Cisco IOS Static Route Syntax

The basic command syntax configured in Global Configuration Mode:

```cisco
Router(config)# ip route <destination_network> <subnet_mask> {next_hop_ip | exit_interface} [administrative_distance]
```

### The 3 Variations:

1. **Next-Hop Route (Recommended for Multi-Access / Ethernet links):**
   ```cisco
   Router(config)# ip route 192.168.20.0 255.255.255.0 10.0.0.2
   ```
   *The router performs a recursive routing table lookup to find the interface to reach `10.0.0.2`.*

2. **Directly Connected Route (Used for Point-to-Point Serial links):**
   ```cisco
   Router(config)# ip route 192.168.20.0 255.255.255.0 Serial0/1/0
   ```
   *Caution on Ethernet:* The router will generate an ARP request for every single destination IP if used on Ethernet interfaces!

3. **Fully Specified Route (Best Practice on Ethernet):**
   ```cisco
   Router(config)# ip route 192.168.20.0 255.255.255.0 GigabitEthernet0/0/1 10.0.0.2
   ```
   *Specifies both the egress interface and the specific next-hop IP address.*

---

## 🌐 The Default Static Route (Gateway of Last Resort)

A **Default Route** (often called the `quad-zero` route) matches all destination IP packets for which no specific route exists in the routing table. It is commonly configured on edge routers pointing to the Internet Service Provider (ISP):

```cisco
Router(config)# ip route 0.0.0.0 0.0.0.0 203.0.113.1
```
- `0.0.0.0` (Destination Network)
- `0.0.0.0` (Subnet Mask = 0 network bits, matching everything)
- `203.0.113.1` (ISP Gateway IP)

---

## 🛡️ Floating Static Route (Redundancy / Backup)

By default, a static route has an **Administrative Distance (AD) of 1**.

A **Floating Static Route** is a backup static route configured with a higher Administrative Distance than the primary route (or dynamic routing protocol like OSPF with AD=110). It remains dormant and invisible in the active routing table until the primary link fails:

```cisco
! Primary route through high-speed Fiber link (AD = 1)
Router(config)# ip route 192.168.50.0 255.255.255.0 10.1.1.2

! Backup floating static route through 4G/LTE cellular backup (AD = 10)
Router(config)# ip route 192.168.50.0 255.255.255.0 10.2.2.2 10
```

If link `10.1.1.2` drops, Cisco IOS instantly installs the backup route `10.2.2.2` into the active routing table!

---

## 🧪 Real-World Two-Router Topology Example

```text
  [ LAN 1 ]                                                          [ LAN 2 ]
192.168.10.0/24                                                   192.168.20.0/24
      │                                                                  │
   (G0/0/0)                                                           (G0/0/0)
   .1 (GW)                                                            .1 (GW)
┌──────────┐            WAN Link: 10.0.0.0/30                      ┌──────────┐
│ Router 1 ├───────────────────────────────────────────────────────┤ Router 2 │
└──────────┘  (G0/0/1) .1                             .2 (G0/0/1)  └──────────┘
```

### Configuration on Router 1:
```cisco
Router1(config)# interface GigabitEthernet0/0/0
Router1(config-if)# ip address 192.168.10.1 255.255.255.0
Router1(config-if)# no shutdown

Router1(config)# interface GigabitEthernet0/0/1
Router1(config-if)# ip address 10.0.0.1 255.255.255.252
Router1(config-if)# no shutdown

! Route to reach remote LAN 2
Router1(config)# ip route 192.168.20.0 255.255.255.0 10.0.0.2
```

### Configuration on Router 2:
```cisco
Router2(config)# interface GigabitEthernet0/0/0
Router2(config-if)# ip address 192.168.20.1 255.255.255.0
Router2(config-if)# no shutdown

Router2(config)# interface GigabitEthernet0/0/1
Router2(config-if)# ip address 10.0.0.2 255.255.255.252
Router2(config-if)# no shutdown

! Route to reach remote LAN 1
Router2(config)# ip route 192.168.10.0 255.255.255.0 10.0.0.1
```

---

## 🛠️ Verification & Troubleshooting Commands

```cisco
! Check IPv4 routing table
Router# show ip route

! View only static routes
Router# show ip route static

! Test end-to-end reachability
Router# ping 192.168.20.1

! Trace the hop-by-hop path taken
Router# traceroute 192.168.20.1
```
