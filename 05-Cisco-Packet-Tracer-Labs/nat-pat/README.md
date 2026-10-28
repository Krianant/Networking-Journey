# 🧪 Lab 06: NAT & PAT (Port Address Translation) Configuration

## 🎯 Objective
- Simulate an Enterprise edge network connecting to the public Internet via an ISP.
- Configure private addressing on the internal LAN (`192.168.1.0/24`).
- Configure an Edge Router with a default static route pointing to the ISP.
- Configure **PAT (NAT Overload)** on the Edge Router using the public WAN interface.
- Configure **Static NAT (Port Forwarding)** to expose an internal HTTP Web Server to the outside world.
- Verify translations using `show ip nat translations` and test external reachability.

---

## 🗺️ Network Topology

```text
  Internal Private LAN (Inside)                                       Simulated Public Internet (Outside)
  
       Staff-PC                                                             Public-DNS-Server
   192.168.1.10 /24                                                           8.8.8.8 /24
          │                                                                        │
       (Fa0/1)                                                                  (Fa0/1)
   ┌─────────────┐                                                          ┌─────────────┐
   │ Switch-LAN  │                                                          │ Switch-ISP  │
   └──────┬──────┘                                                          └──────┬──────┘
          │ (Fa0/24)                                                               │ (Fa0/24)
          │                                                                        │
       (G0/0/0) Inside                                                          (G0/0/1)
   ┌────────────────────────────────┐       Public Link: 203.0.113.0/30      ┌────────────────────────────────┐
   │         Edge Router            ├────────────────────────────────────────┤          ISP Router            │
   │  G0/0/0: 192.168.1.1 (Inside)  │ (G0/0/1) .1                 .2 (G0/0/0)│  G0/0/1: 8.8.8.1 (Outside)     │
   └────────────────────────────────┘                                        └────────────────────────────────┘
          │ (Fa0/2)
       Web-Server
   192.168.1.50 /24
```

---

## 📊 Addressing Table

| Device | Interface | IP Address | Subnet Mask | Default Gateway | NAT Zone |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **Edge-Rtr** | G0/0/0 | `192.168.1.1` | `255.255.255.0` | N/A | `ip nat inside` |
| **Edge-Rtr** | G0/0/1 | `203.0.113.1` | `255.255.255.252` | N/A | `ip nat outside` |
| **ISP-Rtr** | G0/0/0 | `203.0.113.2` | `255.255.255.252` | N/A | Public |
| **ISP-Rtr** | G0/0/1 | `8.8.8.1` | `255.255.255.0` | N/A | Public |
| **Staff-PC** | FastEthernet0 | `192.168.1.10` | `255.255.255.0` | `192.168.1.1` | Inside Local |
| **Web-Server** | FastEthernet0 | `192.168.1.50` | `255.255.255.0` | `192.168.1.1` | Inside Local |
| **Public-DNS** | FastEthernet0 | `8.8.8.8` | `255.255.255.0` | `8.8.8.1` | Outside Global |

---

## ⚙️ Device Configurations

### 1. ISP Router (Simulating Public Infrastructure)
```cisco
enable
configure terminal
hostname ISP-Router

interface GigabitEthernet0/0/0
 description Link-to-Edge-Customer
 ip address 203.0.113.2 255.255.255.252
 no shutdown
exit

interface GigabitEthernet0/0/1
 description Public-Internet-Servers
 ip address 8.8.8.1 255.255.255.0
 no shutdown
exit

! Note: The ISP has NO route to private IP range 192.168.1.0/24!
end
```

### 2. Edge Router Configuration (PAT + Static NAT)
```cisco
enable
configure terminal
hostname Edge-Router

! 1. Configure Interface IPs
interface GigabitEthernet0/0/0
 description Inside-LAN
 ip address 192.168.1.1 255.255.255.0
 ip nat inside
 no shutdown
exit

interface GigabitEthernet0/0/1
 description Outside-ISP
 ip address 203.0.113.1 255.255.255.252
 ip nat outside
 no shutdown
exit

! 2. Default Route to ISP
ip route 0.0.0.0 0.0.0.0 203.0.113.2

! 3. ACL Matching Internal LAN for Internet PAT
ip access-list standard NAT-PERMIT
 permit 192.168.1.0 0.0.0.255
exit

! 4. Dynamic PAT (NAT Overload) using public WAN interface
ip nat inside source list NAT-PERMIT interface GigabitEthernet0/0/1 overload

! 5. Static NAT: Expose Internal Web Server port 80 externally via port 80
ip nat inside source static tcp 192.168.1.50 80 203.0.113.1 80

end
copy running-config startup-config
```

---

## 🔍 Verification & Testing

### 1. Test Outbound PAT from Staff-PC
From **Staff-PC** (`192.168.1.10`):
```cmd
ping 8.8.8.8
```
*Result:* Success! The packet reached the ISP public server and returned.

### 2. Inspect NAT Table on Edge-Router
```cisco
Edge-Router# show ip nat translations
Pro  Inside global          Inside local           Outside local        Outside global
icmp 203.0.113.1:1          192.168.1.10:1         8.8.8.8:1            8.8.8.8:1
tcp  203.0.113.1:80         192.168.1.50:80        ---                  ---
```
- The ICMP ping from `192.168.1.10` was translated into public IP `203.0.113.1`!
- The permanent static mapping for the Web Server (`192.168.1.50:80` $\rightarrow$ `203.0.113.1:80`) is active.

### 3. Test Inbound Web Server Access
From the outside simulated client or browser, open:
```text
http://203.0.113.1
```
The Cisco Packet Tracer web page hosted on internal server `192.168.1.50` renders successfully through the NAT boundary!

### 4. Verify NAT Counters
```cisco
Edge-Router# show ip nat statistics
Total translations: 2 (1 static, 1 dynamic, 1 extended)
Outside Interfaces: GigabitEthernet0/0/1
Inside Interfaces: GigabitEthernet0/0/0
Hits: 38  Misses: 2
```
