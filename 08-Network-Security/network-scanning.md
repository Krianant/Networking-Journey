# 🔍 Network Scanning & Reconnaissance (Nmap Guide)

**Network Scanning** is the process of identifying active hosts on a network, discovering open ports, and fingerprinting running services and operating systems. In cybersecurity and network administration, **Nmap (Network Mapper)** is the industry-standard tool.

---

## 🧭 The 3 Core Stages of Scanning

```text
 ┌──────────────────────┐      ┌──────────────────────┐      ┌──────────────────────┐
 │ 1. Host Discovery    │ ───► │ 2. Port Scanning     │ ───► │ 3. Service/OS Enum   │
 │ (Ping Sweeps)        │      │ (SYN, Connect, UDP)  │      │ (-sV, -O, Nmap NSE)  │
 └──────────────────────┘      └──────────────────────┘      └──────────────────────┘
```

---

## 🛠️ Essential Nmap Scan Types

### 1. TCP SYN Scan (`-sS`) — Default & Most Popular
Also known as **"Half-Open"** or **"Stealth"** scanning.
- Sends a `SYN` packet.
- If target responds with `SYN-ACK`, the port is **OPEN**. Nmap immediately sends an `RST` to tear down the connection without completing the 3-way handshake.
- If target responds with `RST`, the port is **CLOSED**.
- If no response (or ICMP unreachable), the port is **FILTERED** (blocked by firewall).
- *Advantage:* Fast, doesn't complete application sessions (less likely to be logged by simple application logs). Requires root/admin privileges.

### 2. TCP Connect Scan (`-sT`)
- Completes the full 3-way handshake (`SYN` $\rightarrow$ `SYN-ACK` $\rightarrow$ `ACK`).
- Used when the user does not have raw packet creation privileges (non-root users).
- *Disadvantage:* Slower, and logged by almost all modern firewall and server event logs.

### 3. UDP Scan (`-sU`)
- Sends empty UDP packets to target ports (e.g., DNS 53, SNMP 161, DHCP 67).
- If target replies with ICMP Port Unreachable (Type 3, Code 3), port is **CLOSED**.
- If target replies with UDP response, port is **OPEN**.
- If no response, marked as **`open|filtered`**.
- *Note:* Much slower due to OS ICMP rate limiting.

---

## 📋 Port States Defined by Nmap

| Port State | Meaning | What It Indicates to an Analyst |
| :--- | :--- | :--- |
| **`open`** | An application is actively accepting connections on this port. | Attack surface identified; service should be enumerated and patched. |
| **`closed`** | Port is accessible (packet reached host), but no application is listening. | Host is alive, but service is offline. |
| **`filtered`** | Packet was dropped or blocked before reaching the host; Nmap cannot tell if port is open. | Firewall, router ACL, or host firewall (iptables/Windows Defender) is blocking traffic. |
| **`unfiltered`** | Port is accessible, but Nmap cannot determine whether it is open or closed (e.g. ACK scans). | Used to map firewall rulesets. |
| **`open|filtered`** | Nmap cannot determine between open and filtered (common in UDP scans). | Further probing with scripts required. |

---

## ⚡ High-Yield Nmap Command Cheat Sheet

```bash
# 1. Host Discovery: Ping sweep an entire subnet without port scanning
nmap -sn 192.168.1.0/24

# 2. Fast Scan: Scan top 100 most common ports
nmap -F 192.168.1.10

# 3. Scan specific ports or port ranges
nmap -p 22,80,443,8080 192.168.1.10
nmap -p 1-1000 192.168.1.10
nmap -p- 192.168.1.10          # Scan all 65,535 ports!

# 4. Service Version Detection (-sV) & OS Detection (-O)
nmap -sV -O 192.168.1.10

# 5. Aggressive Scan (-A): Enables OS detection, version detection, script scanning, and traceroute
nmap -A 192.168.1.10

# 6. Default Vulnerability & Discovery Script Scan (-sC)
nmap -sC -sV -p 80,443 192.168.1.10

# 7. Scan UDP Ports (DNS, SNMP)
sudo nmap -sU -p 53,161 192.168.1.10

# 8. Output formats (Crucial for lab documentation and penetration test reporting)
nmap -A 192.168.1.10 -oN scan_result.txt   # Normal text format
nmap -A 192.168.1.10 -oX scan_result.xml   # XML format for automated ingestion
```

---

## 🛡️ Defense & SOC Analyst Perspective

When an Nmap scan occurs against your network:
- **Intrusion Detection Systems (Snort / Suricata / Zeek)** flag high volumes of TCP `SYN` packets from a single source address across multiple destination ports within a short window.
- **Firewall Logs:** Show hundreds of denied connection attempts.
- **Defensive Measures:**
  - Implement **Fail2ban** or automated firewall blocking for rapid port scanners.
  - Disable ICMP echo replies on external edge interfaces if stealth is required.
  - Implement restrictive egress and ingress ACLs.
