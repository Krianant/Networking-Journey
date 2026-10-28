# 🌐 My Networking Journey

### From Networking Fundamentals to CCNA-Level Skills

Welcome to my networking learning repository! 🚀

I'm documenting my journey of strengthening my networking fundamentals, building hands-on labs, and developing the practical skills required for a career in **Networking and Cybersecurity**.

My approach is simple: learn the concepts, implement them in a lab, troubleshoot problems, and document what I discover.

This repository serves as my personal networking knowledge base, practical lab portfolio, and record of continuous improvement.

---

## 🎯 My Goals

* Strengthen my understanding of computer networking fundamentals.
* Develop practical skills aligned with the CCNA curriculum.
* Build and document network topologies using Cisco Packet Tracer.
* Understand routing, switching, subnetting, and network troubleshooting.
* Learn network monitoring and packet analysis using Wireshark.
* Apply networking knowledge to cybersecurity and security operations.
* Build a strong foundation for future roles in Network Engineering and Cybersecurity.

---

## 🗺️ My Learning Roadmap

| Phase | Topics                                      | Module / Notes | Status        |
| :---: | :------------------------------------------ | :------------- | :------------ |
| **01** | Networking Fundamentals and OSI Model       | [01-Networking-Fundamentals](./01-Networking-Fundamentals/) | 🟡 Revising   |
| **02** | IPv4 Addressing and Subnetting              | [02-IP-Addressing-Subnetting](./02-IP-Addressing-Subnetting/) | 🟡 Practicing |
| **03** | Ethernet, MAC Addresses, and ARP            | [03-Switching](./03-Switching/) | 🟡 Practicing |
| **04** | Cisco IOS and Basic Switch Configuration    | [04-Routing](./04-Routing/) & [Switch Config](./03-Switching/basic-switch-configuration.md) | 🔜 Next       |
| **05** | VLANs, Trunking, and Inter-VLAN Routing     | [05-Cisco-Packet-Tracer-Labs](./05-Cisco-Packet-Tracer-Labs/) | ⏳ Upcoming    |
| **06** | Static Routing and Default Routes           | [04-Routing](./04-Routing/) | ⏳ Upcoming    |
| **07** | OSPF and Dynamic Routing                    | [04-Routing](./04-Routing/) | ⏳ Upcoming    |
| **08** | DHCP, DNS, NAT, and Network Services        | [01-Networking-Fundamentals](./01-Networking-Fundamentals/common-network-protocols.md) | ⏳ Upcoming    |
| **09** | ACLs and Network Security                   | [08-Network-Security](./08-Network-Security/) | ⏳ Upcoming    |
| **10** | IPv6, Wireless, and Network Management      | [01-Networking-Fundamentals](./01-Networking-Fundamentals/) | ⏳ Upcoming    |
| **11** | Network Troubleshooting and Packet Analysis | [06-Wireshark](./06-Wireshark/) & [07-Troubleshooting](./07-Network-Troubleshooting/) | ⏳ Upcoming    |
| **12** | Advanced Labs and CCNA Practice             | [05-Cisco-Packet-Tracer-Labs](./05-Cisco-Packet-Tracer-Labs/) | ⏳ Upcoming    |

*The status labels reflect my current learning priorities and will be updated as I complete topics and labs.*

---

## 🧪 Hands-On Labs

I believe practical implementation is essential to understanding networking.

My lab collection will include:

* [ ] [Basic LAN configuration](./05-Cisco-Packet-Tracer-Labs/basic-lan/)
* [ ] [IPv4 addressing and subnetting exercises](./05-Cisco-Packet-Tracer-Labs/subnetting-lab/)
* [ ] [Switch MAC address learning](./03-Switching/mac-address-learning.md)
* [ ] [ARP and local network communication](./03-Switching/arp-explained.md)
* [ ] [Default gateway configuration](./04-Routing/default-gateway.md)
* [ ] [Communication between different IP networks](./04-Routing/static-routing.md)
* [ ] [VLAN and trunk configuration](./05-Cisco-Packet-Tracer-Labs/inter-vlan-routing/)
* [ ] [Inter-VLAN routing](./05-Cisco-Packet-Tracer-Labs/inter-vlan-routing/)
* [ ] [Static and default routing](./05-Cisco-Packet-Tracer-Labs/static-routing/)
* [ ] OSPF configuration
* [ ] DHCP and DNS troubleshooting
* [ ] NAT and PAT configuration
* [ ] [Access Control Lists (ACLs)](./08-Network-Security/acl-basics.md)
* [ ] [Wireshark packet analysis](./06-Wireshark/)
* [ ] [Network troubleshooting scenarios](./07-Network-Troubleshooting/)

Each completed lab will include its objective, topology, configuration, verification commands, results, and lessons learned.

---

## 🛠️ Tools and Technologies

| Tool | Purpose |
| :--- | :------ |
| **Cisco Packet Tracer** | Network simulation and configuration |
| **Wireshark** | Packet capture and protocol analysis |
| **Linux** | Networking commands and troubleshooting |
| **Cisco IOS CLI** | Switch and router configuration |
| **Nmap** | Network discovery and service enumeration |

---

## 📚 Topics I'm Currently Strengthening

### 1. IP Addressing and Subnetting
Understanding network IDs, host ranges, broadcast addresses, subnet masks, and CIDR notation.  
📁 *Reference:* [`02-IP-Addressing-Subnetting/`](./02-IP-Addressing-Subnetting/)

### 2. Switching and MAC Address Learning
Learning how switches build MAC address tables and forward Ethernet frames.  
📁 *Reference:* [`03-Switching/`](./03-Switching/)

### 3. ARP and Default Gateways
Understanding how devices resolve local IP addresses to MAC addresses and communicate with devices outside their subnet.  
📁 *Reference:* [`03-Switching/arp-explained.md`](./03-Switching/arp-explained.md) & [`04-Routing/default-gateway.md`](./04-Routing/default-gateway.md)

### 4. Routing Fundamentals
Learning how routers forward packets between different IP networks and how routing tables determine the next hop.  
📁 *Reference:* [`04-Routing/`](./04-Routing/)

### 5. Network Troubleshooting
Practicing a systematic approach using commands such as `ping`, `traceroute`, `ipconfig`, `ip addr`, and `ip route`.  
📁 *Reference:* [`07-Network-Troubleshooting/`](./07-Network-Troubleshooting/)

---

## 📁 Repository Structure

```text
Networking-Journey/
│
├── README.md
│
├── 01-Networking-Fundamentals/
│   ├── osi-model.md
│   ├── tcp-ip-model.md
│   ├── tcp-vs-udp.md
│   └── common-network-protocols.md
│
├── 02-IP-Addressing-Subnetting/
│   ├── ipv4-addressing.md
│   ├── subnetting-practice.md
│   └── cidr-reference.md
│
├── 03-Switching/
│   ├── mac-address-learning.md
│   ├── arp-explained.md
│   └── basic-switch-configuration.md
│
├── 04-Routing/
│   ├── default-gateway.md
│   ├── static-routing.md
│   └── routing-table-analysis.md
│
├── 05-Cisco-Packet-Tracer-Labs/
│   ├── README.md
│   ├── basic-lan/
│   │   └── README.md
│   ├── subnetting-lab/
│   │   └── README.md
│   ├── inter-vlan-routing/
│   │   └── README.md
│   └── static-routing/
│       └── README.md
│
├── 06-Wireshark/
│   ├── icmp-analysis.md
│   ├── dns-analysis.md
│   └── tcp-three-way-handshake.md
│
├── 07-Network-Troubleshooting/
│   ├── connectivity-troubleshooting.md
│   └── troubleshooting-commands.md
│
└── 08-Network-Security/
    ├── acl-basics.md
    ├── network-scanning.md
    └── network-security-fundamentals.md
```

---

## 📈 Learning Philosophy

I don't want to simply memorize networking concepts.

I want to understand:
* Why a packet takes a particular path.
* What happens when two devices communicate.
* How switches and routers make forwarding decisions.
* Why network connectivity fails and how to identify the cause.
* How networking concepts relate to real-world cybersecurity.

> **Learn → Configure → Test → Troubleshoot → Document → Repeat.**

---

## 🚀 Future Direction

My long-term goal is to combine networking expertise with cybersecurity skills.

A strong understanding of networking will help me progress toward areas such as:
* Network Engineering
* Security Operations Center (SOC) Analysis
* Network Security
* Vulnerability Assessment and Penetration Testing
* Cybersecurity Engineering

This repository represents my progress, practical work, and commitment to continuous learning.

---

⭐ Follow my journey as I build, break, troubleshoot, and understand networks one lab at a time.
