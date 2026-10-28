# 🧪 Lab 01: Basic LAN Configuration & Verification

## 🎯 Objective
- Connect two PCs (`PC-A` and `PC-B`) to a Cisco Catalyst 2960 switch (`SW1`).
- Configure IP addressing on the PCs.
- Perform baseline switch management configuration on `SW1` (hostname, passwords, banner, management SVI).
- Test local network connectivity using `ping`.
- Inspect the switch's dynamic MAC address table before and after sending traffic.
- Use Packet Tracer **Simulation Mode** to track ARP and ICMP PDUs.

---

## 🗺️ Network Topology

```text
       PC-A                                          PC-B
 192.168.1.10 /24                              192.168.1.20 /24
        │                                              │
    (FastEthernet0)                                (FastEthernet0)
        │                                              │
        │ [Copper Straight-Through]                    │ [Copper Straight-Through]
        │                                              │
     (Fa0/1)                                        (Fa0/2)
   ┌────────────────────────────────────────────────────────┐
   │                  Cisco 2960 Switch (SW1)               │
   │               Management SVI (VLAN 1):                 │
   │               IP: 192.168.1.2 /24                      │
   └────────────────────────────────────────────────────────┘
```

---

## 📊 Addressing Table

| Device | Interface | IP Address | Subnet Mask | Default Gateway |
| :--- | :--- | :--- | :--- | :--- |
| **SW1** | VLAN 1 (SVI) | `192.168.1.2` | `255.255.255.0` | N/A (Local LAN) |
| **PC-A** | FastEthernet0 | `192.168.1.10` | `255.255.255.0` | `192.168.1.1` (Optional) |
| **PC-B** | FastEthernet0 | `192.168.1.20` | `255.255.255.0` | `192.168.1.1` (Optional) |

---

## ⚙️ Step-by-Step Configuration

### 1. Configure Host IP Addressing in Packet Tracer
- Click **PC-A** $\rightarrow$ **Desktop** tab $\rightarrow$ **IP Configuration**:
  - IP Address: `192.168.1.10`
  - Subnet Mask: `255.255.255.0`
- Click **PC-B** $\rightarrow$ **Desktop** tab $\rightarrow$ **IP Configuration**:
  - IP Address: `192.168.1.20`
  - Subnet Mask: `255.255.255.0`

### 2. Configure SW1 Baseline Configuration
Click **SW1** $\rightarrow$ **CLI** tab and enter:

```cisco
enable
configure terminal

hostname SW1

! Disable DNS lookup
no ip domain-lookup

! Secure Privileged EXEC Mode
enable secret CiscoPass123!

! Secure Console Port
line console 0
 password ConsolePass123!
 login
 logging synchronous
 exec-timeout 5 0
exit

! Secure VTY lines
line vty 0 15
 password VtyPass123!
 login
 logging synchronous
exit

! Legal warning banner
banner motd # AUTHORIZED ACCESS ONLY #

! Configure Management IP Interface
interface vlan 1
 description Switch-Management-Interface
 ip address 192.168.1.2 255.255.255.0
 no shutdown
exit

! Save to NVRAM
end
copy running-config startup-config
```

---

## 🔍 Verification & Testing

### Test 1: Connectivity Test
On **PC-A**, open the Command Prompt (`Desktop` tab $\rightarrow$ `Command Prompt`):
```cmd
ping 192.168.1.20
```
*Expected Output:*
```text
Pinging 192.168.1.20 with 32 bytes of data:
Reply from 192.168.1.20: bytes=32 time<1ms TTL=128
Reply from 192.168.1.20: bytes=32 time<1ms TTL=128
Reply from 192.168.1.20: bytes=32 time<1ms TTL=128
Reply from 192.168.1.20: bytes=32 time<1ms TTL=128

Ping statistics for 192.168.1.20:
    Packets: Sent = 4, Received = 4, Lost = 0 (0% loss)
```

Also verify connectivity to the switch management interface:
```cmd
ping 192.168.1.2
```
*(First ping may time out due to initial ARP resolution; subsequent 3 packets succeed).*

### Test 2: Inspect Switch MAC Address Table
On **SW1 CLI**:
```cisco
SW1# show mac address-table
```
*Sample Output:*
```text
          Mac Address Table
-------------------------------------------

Vlan    Mac Address       Type        Ports
----    -----------       --------    -----
   1    0001.424b.8801    DYNAMIC     Fa0/1
   1    0060.7032.d002    DYNAMIC     Fa0/2
```
SW1 has dynamically learned the hardware MAC addresses of PC-A and PC-B on ports Fa0/1 and Fa0/2!

---

## 🔬 Simulation Mode Observations
1. Switch to **Simulation Mode** (Shift + S).
2. Generate a ping from PC-A to PC-B.
3. Notice two PDUs created at PC-A:
   - Green PDU = **ARP Request** (Broadcast `FF:FF:FF:FF:FF:FF`)
   - Purple PDU = **ICMP Echo Request** (Paused waiting for ARP resolution)
4. Advance the simulation:
   - The switch forwards the ARP request out Fa0/2.
   - PC-B answers with an ARP reply unicast to PC-A.
   - Once PC-A receives the MAC address, the ICMP packet is transmitted!

---

## 💡 Lessons Learned
- Switches operate at Layer 2 and forward based on MAC addresses, not IP addresses.
- Initial pings often drop 1 packet while the switch and PC populate their ARP and MAC address tables.
- An unmanaged switch functions out-of-the-box, but configuring an SVI (VLAN 1) allows remote management and monitoring.
