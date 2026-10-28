# 🧪 Lab 02: Subnetting & Addressing Implementation

## 🎯 Objective
- Take the private base network `192.168.10.0 /24` and design a VLSM subnetting scheme.
- Segment into three subnets:
  - **Subnet A (Staff LAN):** Requires 50 hosts.
  - **Subnet B (Guest LAN):** Requires 25 hosts.
  - **Subnet C (Management LAN):** Requires 10 hosts.
- Configure router GigabitEthernet interfaces as the Default Gateways for each subnet.
- Configure client hosts and verify intra-subnet and inter-subnet routing.

---

## 🧮 VLSM Addressing Scheme Design

Base Network: `192.168.10.0 /24`

| Subnet Name | Hosts Needed | Formula: $2^H - 2$ | Prefix | Subnet Mask | Allocated Network | Usable Host Range | Broadcast |
| :--- | :---: | :---: | :---: | :--- | :--- | :--- | :--- |
| **Staff LAN** | 50 | $2^6 - 2 = 62$ | **/26** | `255.255.255.192` | `192.168.10.0/26` | `192.168.10.1` – `192.168.10.62` | `192.168.10.63` |
| **Guest LAN** | 25 | $2^5 - 2 = 30$ | **/27** | `255.255.255.224` | `192.168.10.64/27` | `192.168.10.65` – `192.168.10.94` | `192.168.10.95` |
| **Management**| 10 | $2^4 - 2 = 14$ | **/28** | `255.255.255.240` | `192.168.10.96/28` | `192.168.10.97` – `192.168.10.110` | `192.168.10.111` |

---

## 🗺️ Network Topology

```text
       Staff-PC                               Guest-PC
   192.168.10.10 /26                      192.168.10.70 /27
          │                                      │
       (Fa0/1)                                (Fa0/1)
   ┌─────────────┐                        ┌─────────────┐
   │ SW-Staff    │                        │ SW-Guest    │
   └──────┬──────┘                        └──────┬──────┘
          │ (Fa0/24)                             │ (Fa0/24)
          │                                      │
       (G0/0/0)                               (G0/0/1)
   ┌────────────────────────────────────────────────────┐
   │                 Cisco 4321 Router (R1)             │
   │  G0/0/0: 192.168.10.1 /26 (Staff Gateway)          │
   │  G0/0/1: 192.168.10.65 /27 (Guest Gateway)         │
   └────────────────────────────────────────────────────┘
```

---

## 📊 Addressing Table

| Device | Interface | IP Address | Subnet Mask | Default Gateway |
| :--- | :--- | :--- | :--- | :--- |
| **R1** | G0/0/0 | `192.168.10.1` | `255.255.255.192` | N/A |
| **R1** | G0/0/1 | `192.168.10.65` | `255.255.255.224` | N/A |
| **Staff-PC** | FastEthernet0 | `192.168.10.10` | `255.255.255.192` | `192.168.10.1` |
| **Guest-PC** | FastEthernet0 | `192.168.10.70` | `255.255.255.224` | `192.168.10.65` |

---

## ⚙️ Router Configuration (R1)

```cisco
enable
configure terminal
hostname R1

! Configure Staff Gateway Interface
interface GigabitEthernet0/0/0
 description Staff-LAN-Gateway
 ip address 192.168.10.1 255.255.255.192
 no shutdown
exit

! Configure Guest Gateway Interface
interface GigabitEthernet0/0/1
 description Guest-LAN-Gateway
 ip address 192.168.10.65 255.255.255.224
 no shutdown
exit

end
copy running-config startup-config
```

---

## 🔍 Verification & Testing

### 1. Test Router Interface Status
```cisco
R1# show ip interface brief
Interface              IP-Address      OK? Method Status                Protocol
GigabitEthernet0/0/0   192.168.10.1    YES manual up                    up
GigabitEthernet0/0/1   192.168.10.65   YES manual up                    up
```

### 2. Verify Routing Table on R1
```cisco
R1# show ip route connected
C    192.168.10.0/26 is directly connected, GigabitEthernet0/0/0
C    192.168.10.64/27 is directly connected, GigabitEthernet0/0/1
```
*R1 automatically routes between directly connected interfaces!*

### 3. End-to-End Inter-Subnet Ping Test
From **Staff-PC** (`192.168.10.10`):
```cmd
:: 1. Ping Local Default Gateway
ping 192.168.10.1

:: 2. Ping Remote Subnet Gateway
ping 192.168.10.65

:: 3. Ping Remote Host on Guest LAN
ping 192.168.10.70
```
All tests succeed with 0% packet loss!

---

## 💡 Troubleshooting Reflection
- **Common Mistake:** Assigning an IP outside the usable range (e.g. attempting to assign `192.168.10.63` to a PC returns an invalid IP error in Packet Tracer because `.63` is the broadcast address for the `/26` subnet).
- **Subnet Mask Misconfiguration:** If a host is configured with `/24` (`255.255.255.0`) instead of `/26` (`255.255.255.192`), it assumes `192.168.10.70` is on its local subnet and attempts ARP directly instead of sending to the default gateway, resulting in communication failure.
