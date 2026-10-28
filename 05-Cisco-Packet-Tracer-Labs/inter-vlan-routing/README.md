# 🧪 Lab 03: Inter-VLAN Routing (Router-on-a-Stick)

## 🎯 Objective
- Create separate Virtual Local Area Networks (VLANs) on a Cisco Catalyst 2960 switch:
  - **VLAN 10 (Sales)**: Subnet `192.168.10.0/24`
  - **VLAN 20 (Engineering)**: Subnet `192.168.20.0/24`
- Configure switch access ports for end devices.
- Configure an **802.1Q Trunk Link** between the switch and router.
- Configure **Router-on-a-Stick (ROAS)** sub-interfaces on the router to enable inter-VLAN routing.
- Verify communication and traffic isolation between VLANs.

---

## 🗺️ Network Topology

```text
       PC-Sales (VLAN 10)                     PC-Eng (VLAN 20)
       192.168.10.10 /24                      192.168.20.10 /24
               │                                      │
           (Fa0/10)                                (Fa0/20)
       ┌──────────────────────────────────────────────────────┐
       │                Cisco 2960 Switch (SW1)               │
       └──────────────────────────┬───────────────────────────┘
                                  │ (G0/1) 802.1Q Trunk
                                  │
                                  │ (G0/0/0) Trunk Link
       ┌──────────────────────────┴───────────────────────────┐
       │                Cisco 4321 Router (R1)                │
       │  Sub-interface G0/0/0.10 -> 192.168.10.1 (VLAN 10)   │
       │  Sub-interface G0/0/0.20 -> 192.168.20.1 (VLAN 20)   │
       └──────────────────────────────────────────────────────┘
```

---

## 📊 Addressing Table

| Device | Interface | IP Address | Subnet Mask | Default Gateway | VLAN Membership |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **R1** | G0/0/0 | Unnumbered | N/A | N/A | Physical Trunk |
| **R1** | G0/0/0.10 | `192.168.10.1` | `255.255.255.0` | N/A | VLAN 10 (802.1Q) |
| **R1** | G0/0/0.20 | `192.168.20.1` | `255.255.255.0` | N/A | VLAN 20 (802.1Q) |
| **SW1** | VLAN 1 | `192.168.10.2` | `255.255.255.0` | `192.168.10.1` | Management |
| **PC-Sales** | Fa0 | `192.168.10.10` | `255.255.255.0` | `192.168.10.1` | VLAN 10 Access |
| **PC-Eng** | Fa0 | `192.168.20.10` | `255.255.255.0` | `192.168.20.1` | VLAN 20 Access |

---

## ⚙️ Device Configurations

### 1. Switch Configuration (SW1)
```cisco
enable
configure terminal
hostname SW1

! Create VLANs
vlan 10
 name Sales
exit

vlan 20
 name Engineering
exit

! Assign Access Port for Sales PC
interface FastEthernet0/10
 description PC-Sales-VLAN10
 switchport mode access
 switchport access vlan 10
 no shutdown
exit

! Assign Access Port for Engineering PC
interface FastEthernet0/20
 description PC-Eng-VLAN20
 switchport mode access
 switchport access vlan 20
 no shutdown
exit

! Configure Trunk Port to Router
interface GigabitEthernet0/1
 description Trunk-to-Router-R1
 switchport mode trunk
 switchport trunk allowed vlan 10,20
 no shutdown
exit

end
copy running-config startup-config
```

### 2. Router Configuration (R1 - Router-on-a-Stick)
```cisco
enable
configure terminal
hostname R1

! Enable physical interface (DO NOT assign an IP address to the parent interface!)
interface GigabitEthernet0/0/0
 no ip address
 no shutdown
exit

! Configure Sub-interface for VLAN 10
interface GigabitEthernet0/0/0.10
 description Gateway-for-VLAN10-Sales
 encapsulation dot1Q 10
 ip address 192.168.10.1 255.255.255.0
exit

! Configure Sub-interface for VLAN 20
interface GigabitEthernet0/0/0.20
 description Gateway-for-VLAN20-Engineering
 encapsulation dot1Q 20
 ip address 192.168.20.1 255.255.255.0
exit

end
copy running-config startup-config
```

> ⚠️ **Critical Rule:** In Cisco IOS, you **must** issue `encapsulation dot1Q <vlan-id>` **before** you can assign an IP address to a sub-interface!

---

## 🔍 Verification & Testing

### 1. Verify VLAN and Trunking on SW1
```cisco
SW1# show vlan brief
VLAN Name                             Status    Ports
---- -------------------------------- --------- -------------------------------
1    default                          active    Fa0/1 - Fa0/9, Fa0/11 - Fa0/19...
10   Sales                            active    Fa0/10
20   Engineering                      active    Fa0/20

SW1# show interfaces trunk
Port        Mode             Encapsulation  Status        Native vlan
Gi0/1       on               802.1q         trunking      1

Port        Vlans allowed on trunk
Gi0/1       10,20
```

### 2. Verify Sub-interfaces on R1
```cisco
R1# show ip interface brief
Interface              IP-Address      OK? Method Status                Protocol
GigabitEthernet0/0/0   unassigned      YES unset  up                    up
GigabitEthernet0/0/0.10 192.168.10.1   YES manual up                    up
GigabitEthernet0/0/0.20 192.168.20.1   YES manual up                    up
```

### 3. Inter-VLAN Ping Test
From **PC-Sales** (`192.168.10.10`):
```cmd
ping 192.168.20.10
```
*Result:* Success! The packet travels:
`PC-Sales` $\rightarrow$ `SW1 (tagged VLAN 10)` $\rightarrow$ `R1 G0/0/0.10` $\rightarrow$ *Routed by R1* $\rightarrow$ `R1 G0/0/0.20 (tagged VLAN 20)` $\rightarrow$ `SW1` $\rightarrow$ `PC-Eng`.

---

## 💡 Key Takeaways
- Without a Layer 3 device (router or L3 switch), VLANs are completely isolated broadcast domains and cannot communicate.
- In Router-on-a-Stick, a single physical cable carries tagged frames for multiple VLANs over an 802.1Q trunk link.
- In high-throughput corporate environments, ROAS can become a bottleneck (single physical link saturation); Layer 3 Switches using **Switch Virtual Interfaces (SVIs)** are preferred for wire-speed performance.
