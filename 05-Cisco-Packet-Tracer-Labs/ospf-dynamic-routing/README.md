# 🧪 Lab 05: Single-Area OSPFv2 Dynamic Routing

## 🎯 Objective
- Build a 3-router meshed topology in Cisco Packet Tracer.
- Configure IPv4 addressing and Loopback interfaces on all routers.
- Enable **OSPFv2 (Single-Area, Area 0)** across all routers.
- Configure custom **Router IDs** and modify **Reference Bandwidth** for Gigabit links.
- Configure **Passive Interfaces** to secure local LANs and prevent unnecessary Hello multicasting.
- Verify neighbor adjacencies (`show ip ospf neighbor`) and route convergence.
- Test route failover by shutting down a primary link.

---

## 🗺️ Network Topology

```text
       Staff LAN 1                                                    Staff LAN 2
     192.168.10.0/24                                                192.168.20.0/24
            │                                                              │
         (G0/0/0)                                                       (G0/0/0)
      ┌───────────┐           Link R1-R2: 10.1.2.0/30 (G0/0/1)        ┌───────────┐
      │    R1     ├───────────────────────────────────────────────────┤    R2     │
      └─────┬─────┘                                                   └─────┬─────┘
            │ (G0/0/2)                                                      │ (G0/0/2)
            │                                                               │
            │ Link R1-R3: 10.1.3.0/30           Link R2-R3: 10.2.3.0/30     │
            └─────────────────────────┌───────────┐─────────────────────────┘
                                      │    R3     │
                                      └─────┬─────┘
                                            │ (G0/0/0)
                                            │
                                     Server Farm LAN
                                     192.168.30.0/24
```

---

## 📊 Addressing Table

| Device | Interface | IP Address | Subnet Mask | Description |
| :--- | :--- | :--- | :--- | :--- |
| **R1** | Loopback0 | `1.1.1.1` | `255.255.255.255` | OSPF Router-ID |
| **R1** | G0/0/0 | `192.168.10.1` | `255.255.255.0` | Staff LAN 1 Gateway |
| **R1** | G0/0/1 | `10.1.2.1` | `255.255.255.252` | Link to R2 |
| **R1** | G0/0/2 | `10.1.3.1` | `255.255.255.252` | Link to R3 |
| **R2** | Loopback0 | `2.2.2.2` | `255.255.255.255` | OSPF Router-ID |
| **R2** | G0/0/0 | `192.168.20.1` | `255.255.255.0` | Staff LAN 2 Gateway |
| **R2** | G0/0/1 | `10.1.2.2` | `255.255.255.252` | Link to R1 |
| **R2** | G0/0/2 | `10.2.3.1` | `255.255.255.252` | Link to R3 |
| **R3** | Loopback0 | `3.3.3.3` | `255.255.255.255` | OSPF Router-ID |
| **R3** | G0/0/0 | `192.168.30.1` | `255.255.255.0` | Server Farm Gateway |
| **R3** | G0/0/1 | `10.1.3.2` | `255.255.255.252` | Link to R1 |
| **R3** | G0/0/2 | `10.2.3.2` | `255.255.255.252` | Link to R2 |

---

## ⚙️ Device Configurations

### 1. Router 1 (R1)
```cisco
enable
configure terminal
hostname R1

interface Loopback0
 ip address 1.1.1.1 255.255.255.255
exit

interface GigabitEthernet0/0/0
 description LAN-Staff-1
 ip address 192.168.10.1 255.255.255.0
 no shutdown
exit

interface GigabitEthernet0/0/1
 description WAN-to-R2
 ip address 10.1.2.1 255.255.255.252
 no shutdown
exit

interface GigabitEthernet0/0/2
 description WAN-to-R3
 ip address 10.1.3.1 255.255.255.252
 no shutdown
exit

! OSPF Configuration
router ospf 1
 router-id 1.1.1.1
 auto-cost reference-bandwidth 10000
 passive-interface GigabitEthernet0/0/0
 network 1.1.1.1 0.0.0.0 area 0
 network 192.168.10.0 0.0.0.255 area 0
 network 10.1.2.0 0.0.0.3 area 0
 network 10.1.3.0 0.0.0.3 area 0
exit

end
copy running-config startup-config
```

### 2. Router 2 (R2)
```cisco
enable
configure terminal
hostname R2

interface Loopback0
 ip address 2.2.2.2 255.255.255.255
exit

interface GigabitEthernet0/0/0
 description LAN-Staff-2
 ip address 192.168.20.1 255.255.255.0
 no shutdown
exit

interface GigabitEthernet0/0/1
 description WAN-to-R1
 ip address 10.1.2.2 255.255.255.252
 no shutdown
exit

interface GigabitEthernet0/0/2
 description WAN-to-R3
 ip address 10.2.3.1 255.255.255.252
 no shutdown
exit

! OSPF Configuration
router ospf 1
 router-id 2.2.2.2
 auto-cost reference-bandwidth 10000
 passive-interface GigabitEthernet0/0/0
 network 2.2.2.2 0.0.0.0 area 0
 network 192.168.20.0 0.0.0.255 area 0
 network 10.1.2.0 0.0.0.3 area 0
 network 10.2.3.0 0.0.0.3 area 0
exit

end
copy running-config startup-config
```

### 3. Router 3 (R3)
```cisco
enable
configure terminal
hostname R3

interface Loopback0
 ip address 3.3.3.3 255.255.255.255
exit

interface GigabitEthernet0/0/0
 description LAN-Servers
 ip address 192.168.30.1 255.255.255.0
 no shutdown
exit

interface GigabitEthernet0/0/1
 description WAN-to-R1
 ip address 10.1.3.2 255.255.255.252
 no shutdown
exit

interface GigabitEthernet0/0/2
 description WAN-to-R2
 ip address 10.2.3.2 255.255.255.252
 no shutdown
exit

! OSPF Configuration
router ospf 1
 router-id 3.3.3.3
 auto-cost reference-bandwidth 10000
 passive-interface GigabitEthernet0/0/0
 network 3.3.3.3 0.0.0.0 area 0
 network 192.168.30.0 0.0.0.255 area 0
 network 10.1.3.0 0.0.0.3 area 0
 network 10.2.3.0 0.0.0.3 area 0
exit

end
copy running-config startup-config
```

---

## 🔍 Verification & Failover Testing

### 1. Check Neighbor Adjacency
On **R1**:
```cisco
R1# show ip ospf neighbor
Neighbor ID     Pri   State           Dead Time   Address         Interface
2.2.2.2           1   FULL/BDR        00:00:34    10.1.2.2        GigabitEthernet0/0/1
3.3.3.3           1   FULL/BDR        00:00:32    10.1.3.2        GigabitEthernet0/0/2
```
*Both neighbors are in `FULL` state!*

### 2. Check OSPF Learned Routes
```cisco
R1# show ip route ospf
O        192.168.20.0/24 [110/11] via 10.1.2.2, GigabitEthernet0/0/1
O        192.168.30.0/24 [110/11] via 10.1.3.2, GigabitEthernet0/0/2
O        10.2.3.0/30     [110/20] via 10.1.2.2, GigabitEthernet0/0/1
                         [110/20] via 10.1.3.2, GigabitEthernet0/0/2
```

### 3. Dynamic Convergence & Link Failover Test
On **R1**, shut down the primary link to R2:
```cisco
R1(config)# interface GigabitEthernet0/0/1
R1(config-if)# shutdown
```

Now check the route to LAN 2 on R1:
```cisco
R1# show ip route 192.168.20.0
Routing entry for 192.168.20.0/24
  Known via "ospf 1", distance 110, metric 21
  Routing Descriptor Blocks:
  * 10.1.3.2, via GigabitEthernet0/0/2
```
*OSPF dynamically rerouted traffic to LAN 2 through R3 (`10.1.3.2`) in less than 5 seconds with zero packet loss after convergence!*
