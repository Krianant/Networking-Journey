# 🌐 Network Address Translation (NAT) & Port Address Translation (PAT)

**Network Address Translation (NAT)**, defined in RFC 3022, translates private IPv4 addresses (RFC 1918) into globally unique public IPv4 addresses so private internal hosts can access resources across the public Internet.

---

## 🏷️ The 4 NAT Address Terminology Types

Understanding NAT requires understanding Cisco's 4 address terms from the perspective of the translation device:

```text
       [ Inside Local ]                                  [ Outside Local ]
   192.168.1.50 (Host PC)                                203.0.113.80 (Web Server)
          │                                                       │
   (G0/0/0 Inside)                                         (G0/0/1 Outside)
   ┌──────────────────────────────────────────────────────────────┐
   │                     Edge NAT Router                          │
   └──────────────────────────────────────────────────────────────┘
          │                                                       │
       [ Inside Global ]                                 [ Outside Global ]
   198.51.100.1 (Public IP of Router)                    203.0.113.80 (Public IP of Web Server)
```

| Term | Definition | Example in Diagram |
| :--- | :--- | :--- |
| **Inside Local** | The private IP address configured on the internal host inside the network. | `192.168.1.50` |
| **Inside Global** | The legitimate, globally routable public IP address representing the inside host to the outside world. | `198.51.100.1` |
| **Outside Local** | The IP address of an outside host as known to hosts on the inside network (usually identical to Outside Global). | `203.0.113.80` |
| **Outside Global** | The actual public IP address assigned to a host located on the outside Internet. | `203.0.113.80` |

---

## 🔍 The 3 Types of NAT

### 1. Static NAT (One-to-One)
Maps one private IP address to one permanent public IP address.
- **Use Case:** Exposing an internal server (Web Server, FTP, Mail) to the public Internet with a fixed public IP.
- **Bidirectional:** Connections can be initiated from both outside and inside.

```cisco
! Map private internal server 192.168.1.100 to public IP 198.51.100.10
Router(config)# ip nat inside source static 192.168.1.100 198.51.100.10

! Define Inside and Outside interfaces
Router(config)# interface GigabitEthernet0/0/0
Router(config-if)# description Inside-Private-LAN
Router(config-if)# ip nat inside
Router(config-if)# exit

Router(config)# interface GigabitEthernet0/0/1
Router(config-if)# description Outside-Public-WAN
Router(config-if)# ip nat outside
Router(config-if)# exit
```

---

### 2. Dynamic NAT (Pool of Public IPs)
Translates private IP addresses to a pool of public IP addresses on a first-come, first-served basis.
- If all public IPs in the pool are in use, subsequent hosts must wait until a translation times out.
- Rarely used in modern environments compared to PAT.

```cisco
! 1. Define standard ACL matching allowed internal hosts
Router(config)# access-list 1 permit 192.168.10.0 0.0.0.255

! 2. Define pool of available public IP addresses
Router(config)# ip nat pool PUBLIC-POOL 198.51.100.10 198.51.100.20 netmask 255.255.255.0

! 3. Bind ACL to the Public Pool
Router(config)# ip nat inside source list 1 pool PUBLIC-POOL

! 4. Assign interfaces
Router(config)# interface GigabitEthernet0/0/0
Router(config-if)# ip nat inside
Router(config)# interface GigabitEthernet0/0/1
Router(config-if)# ip nat outside
```

---

### 3. Port Address Translation (PAT / NAT Overload) — Most Common!
Maps thousands of private internal IP addresses to a **single public IP address** (or a small pool) by assigning a unique Layer 4 source port number to each connection stream.
- **Capacity:** Can support ~64,000 simultaneous connections per public IP (limited by available 16-bit ephemeral ports).
- **Home & Enterprise Standard:** Used on virtually all home Wi-Fi routers and corporate egress firewalls.

#### Configuration Option A: Overload using the Router's Outgoing Public Interface IP
*(Standard when ISP provides a dynamic public IP via DHCP or PPPoE):*

```cisco
! 1. Create ACL matching private internal subnets allowed Internet access
Router(config)# ip access-list standard NAT-PERMITTED-HOSTS
Router(config-std-nacl)# permit 192.168.10.0 0.0.0.255
Router(config-std-nacl)# permit 192.168.20.0 0.0.0.255
Router(config-std-nacl)# exit

! 2. Configure PAT using the 'overload' keyword on egress interface
Router(config)# ip nat inside source list NAT-PERMITTED-HOSTS interface GigabitEthernet0/0/1 overload

! 3. Designate NAT inside and outside interfaces
Router(config)# interface GigabitEthernet0/0/0
Router(config-if)# ip nat inside
Router(config-if)# exit

Router(config)# interface GigabitEthernet0/0/1
Router(config-if)# ip nat outside
Router(config-if)# exit
```

#### Configuration Option B: Port Forwarding with PAT (Static PAT)
Expose an internal Web Server (`192.168.10.50:80`) to the world through the router's single public IP on port `8080`:

```cisco
Router(config)# ip nat inside source static tcp 192.168.10.50 80 interface GigabitEthernet0/0/1 8080
```

---

## 🛠️ Verification & Troubleshooting Commands

```cisco
! 1. View active NAT translation table
Router# show ip nat translations

! 2. View NAT statistics (hits, misses, active translations, pool usage)
Router# show ip nat statistics

! 3. Clear dynamic NAT translations from memory
Router# clear ip nat translation *

! 4. Real-time NAT translation debugging (Use carefully!)
Router# debug ip nat
Router# undebug all
```

### Sample Output: `show ip nat translations`
```text
Pro  Inside global          Inside local           Outside local        Outside global
tcp  198.51.100.1:49152     192.168.10.15:49152    142.250.190.46:443   142.250.190.46:443
tcp  198.51.100.1:49153     192.168.10.20:49152    93.184.216.34:80     93.184.216.34:80
udp  198.51.100.1:53210     192.168.10.15:53210    8.8.8.8:53           8.8.8.8:53
```
*(Notice how both `192.168.10.15` and `192.168.10.20` share the single inside global IP `198.51.100.1` differentiated by their port numbers!)*

---

## ⚠️ Common NAT Troubleshooting Pitfalls

1. **Forgot `ip nat inside` or `ip nat outside`:** The router will route the packet normally with its private source IP; the ISP drops it.
2. **Missing `overload` keyword:** Without `overload`, Cisco IOS treats it as dynamic NAT and stops translating once the single interface IP is used by the first packet!
3. **ACL Misconfiguration:** ACL does not match the internal host's subnet.
4. **Missing Default Route:** The router translates the packet, but doesn't know where to forward it. Always ensure an active default route exists:
   ```cisco
   Router(config)# ip route 0.0.0.0 0.0.0.0 GigabitEthernet0/0/1
   ```
