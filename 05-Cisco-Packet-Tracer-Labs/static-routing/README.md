# 🧪 Lab 04: Multi-Router Static Routing & Redundancy

## 🎯 Objective
- Build a 3-router enterprise branch network in Packet Tracer.
- Configure IPv4 addressing and loopback interfaces.
- Configure static routes for full end-to-end network reachability.
- Configure a default static route on the branch router.
- Implement a **Floating Static Route** for automatic link failover and test redundancy.

---

## 🗺️ Network Topology

```text
    Branch LAN                                                        HQ LAN
  192.168.10.0/24                                                 192.168.30.0/24
         │                                                               │
      (G0/0/0)                                                        (G0/0/0)
   ┌───────────┐           Primary Link: 10.1.1.0/30 (G0/0/1)       ┌───────────┐
   │ R1-Branch ├────────────────────────────────────────────────────┤   R2-HQ   │
   └─────┬─────┘                                                    └─────┬─────┘
         │ (G0/0/2)                                                       │ (G0/0/1)
         │                                                                │
         │                  Backup Link: 10.2.2.0/30                      │
         └─────────────────────────┌───────────┐──────────────────────────┘
                                   │  R3-Backup│
                                   └───────────┘
```

---

## 📊 Addressing Table

| Device | Interface | IP Address | Subnet Mask | Description |
| :--- | :--- | :--- | :--- | :--- |
| **R1-Branch** | G0/0/0 | `192.168.10.1` | `255.255.255.0` | Branch Local LAN Gateway |
| **R1-Branch** | G0/0/1 | `10.1.1.1` | `255.255.255.252` | Primary Link to R2 |
| **R1-Branch** | G0/0/2 | `10.2.2.1` | `255.255.255.252` | Backup Link to R3 |
| **R2-HQ** | G0/0/0 | `192.168.30.1` | `255.255.255.0` | HQ Local LAN Gateway |
| **R2-HQ** | G0/0/1 | `10.1.1.2` | `255.255.255.252` | Primary Link to R1 |
| **PC-Branch** | NIC | `192.168.10.10` | `255.255.255.0` | Default Gateway: `192.168.10.1` |
| **Server-HQ** | NIC | `192.168.30.50` | `255.255.255.0` | Default Gateway: `192.168.30.1` |

---

## ⚙️ Device Configurations

### 1. R1-Branch Configuration
```cisco
enable
configure terminal
hostname R1-Branch

interface GigabitEthernet0/0/0
 ip address 192.168.10.1 255.255.255.0
 no shutdown
exit

interface GigabitEthernet0/0/1
 ip address 10.1.1.1 255.255.255.252
 no shutdown
exit

interface GigabitEthernet0/0/2
 ip address 10.2.2.1 255.255.255.252
 no shutdown
exit

! 1. Primary Default Route pointing to R2 via high-speed primary link (AD = 1)
ip route 0.0.0.0 0.0.0.0 10.1.1.2

! 2. Floating Backup Default Route pointing to R3 (AD = 10)
ip route 0.0.0.0 0.0.0.0 10.2.2.2 10

end
copy running-config startup-config
```

### 2. R2-HQ Configuration
```cisco
enable
configure terminal
hostname R2-HQ

interface GigabitEthernet0/0/0
 ip address 192.168.30.1 255.255.255.0
 no shutdown
exit

interface GigabitEthernet0/0/1
 ip address 10.1.1.2 255.255.255.252
 no shutdown
exit

! Static Route back to Branch LAN
ip route 192.168.10.0 255.255.255.0 10.1.1.1

end
copy running-config startup-config
```

---

## 🔍 Verification & Failover Testing

### Step 1: Verify Initial Routing Table
On **R1-Branch**:
```cisco
R1-Branch# show ip route static
S*    0.0.0.0/0 [1/0] via 10.1.1.2
```
*Note that the floating static route (`10.2.2.2` with AD=10) does NOT appear because the primary route (AD=1) is active!*

### Step 2: Test End-to-End Connectivity
From **PC-Branch**:
```cmd
tracert 192.168.30.50
```
*Output:*
```text
  1   <1 ms    192.168.10.1 (R1 Gateway)
  2    1 ms    10.1.1.2 (R2 HQ via Primary Link)
  3    1 ms    192.168.30.50 (Server-HQ)
```

### Step 3: Simulate Link Failure & Failover
Shut down the primary link on R1-Branch:
```cisco
R1-Branch(config)# interface GigabitEthernet0/0/1
R1-Branch(config-if)# shutdown
```

Inspect the routing table on R1-Branch:
```cisco
R1-Branch# show ip route static
S*    0.0.0.0/0 [10/0] via 10.2.2.2
```
*The floating static route (`[10/0] via 10.2.2.2`) was instantly installed!*

Re-run traceroute from PC-Branch:
```cmd
tracert 192.168.30.50
```
*Output:*
```text
  1   <1 ms    192.168.10.1
  2    2 ms    10.2.2.2 (R3 Backup Route)
  ...
```

Restore the primary link:
```cisco
R1-Branch(config)# interface GigabitEthernet0/0/1
R1-Branch(config-if)# no shutdown
```
The primary route with AD=1 immediately preempts the backup route.
