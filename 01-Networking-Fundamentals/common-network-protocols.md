# 📑 Common Network Protocols & Port Numbers

A comprehensive reference for the most essential protocols encountered in CCNA studies, network administration, packet analysis, and cybersecurity operations.

---

## 🔢 Port Ranges Overview

- **Well-Known Ports:** `0 – 1023` (Reserved for system/core network services)
- **Registered Ports:** `1024 – 49151` (Vendor and user applications)
- **Dynamic / Ephemeral / Private Ports:** `49152 – 65535` (Temporary outbound ports assigned by OS)

---

## 📋 Essential Protocols Cheat Sheet

| Port | Protocol | Transport | Full Name | Primary Function & Notes |
| :---: | :--- | :---: | :--- | :--- |
| **20, 21** | **FTP** | TCP | File Transfer Protocol | Port 21 for command/control, Port 20 for data. *Cleartext*. |
| **22** | **SSH** / **SFTP** | TCP | Secure Shell | Encrypted remote terminal & secure file transfer. Replaced Telnet. |
| **23** | **Telnet** | TCP | Teletype Network | Remote terminal CLI. **Insecure** (sends passwords in cleartext). |
| **25** | **SMTP** | TCP | Simple Mail Transfer Protocol | Sends email between servers. (Port 587/465 used with TLS/SSL). |
| **53** | **DNS** | UDP / TCP | Domain Name System | Resolves FQDNs to IPs. UDP for standard queries, TCP for zone transfers/large responses (>512 bytes). |
| **67, 68** | **DHCP** | UDP | Dynamic Host Config Protocol | Port 67 (Server), Port 68 (Client). Automates IP configuration (DORA process). |
| **69** | **TFTP** | UDP | Trivial File Transfer Protocol | Simple, lightweight file transfer without authentication. Used for Cisco IOS backups and PXE boot. |
| **80** | **HTTP** | TCP | Hypertext Transfer Protocol | Unencrypted World Wide Web traffic. |
| **88** | **Kerberos** | TCP / UDP | Kerberos Authentication | Windows Active Directory authentication service. |
| **110** | **POP3** | TCP | Post Office Protocol v3 | Retrieves email from server to client. Insecure (Port 995 for POP3S). |
| **123** | **NTP** | UDP | Network Time Protocol | Synchronizes system clocks across network devices. Critical for log correlation. |
| **143** | **IMAP** | TCP | Internet Message Access Protocol | Modern email retrieval with server-side synchronization (Port 993 for IMAPS). |
| **161, 162** | **SNMP** | UDP | Simple Network Management Protocol | Port 161 (Queries/Polls), Port 162 (SNMP Traps). Monitoring network devices. (SNMPv3 recommended for encryption). |
| **389** | **LDAP** | TCP / UDP | Lightweight Directory Access Protocol | Directory querying (Active Directory). Insecure (Port 636 for LDAPS). |
| **443** | **HTTPS** | TCP | HTTP Secure (HTTP over TLS) | Encrypted web browsing via TLS/SSL. |
| **445** | **SMB** | TCP | Server Message Block | Windows file/printer sharing. Frequent target for lateral movement exploits (e.g. EternalBlue). |
| **514** | **Syslog** | UDP | System Logging Protocol | Standard device log aggregation to SIEM or syslog servers. |
| **3389** | **RDP** | TCP / UDP | Remote Desktop Protocol | Microsoft Windows Remote Desktop GUI management. |

---

## 🔄 Core Network Infrastructure Protocols (Non-Port / Network Layer)

| Protocol | Layer | Protocol # / Type | Description |
| :--- | :---: | :---: | :--- |
| **ICMP** | Layer 3 | IP Protocol `1` | Diagnostics and error messaging (`ping`, `traceroute`). |
| **IGMP** | Layer 3 | IP Protocol `2` | Multicast group membership management for IPv4. |
| **OSPF** | Layer 3 | IP Protocol `89` | Open Shortest Path First link-state routing protocol. |
| **EIGRP** | Layer 3 | IP Protocol `88` | Cisco proprietary/advanced distance-vector routing protocol. |
| **ARP** | Layer 2 | EtherType `0x0806` | Resolves known IPv4 addresses to MAC addresses. |

---

## 🔒 Security Best Practices for Network Ports

1. **Disable Insecure Legacy Protocols:**
   - Replace **Telnet (23)** with **SSH (22)**.
   - Replace **HTTP (80)** with **HTTPS (443)** using HSTS.
   - Replace **FTP (21)** with **SFTP (22)** or **FTPS**.
   - Use **SNMPv3** with authentication and privacy (`authPriv`) over SNMPv1/v2c.
2. **Close Unused Ports:** Follow the principle of least privilege on edge firewalls and access switches.
3. **Internal Segmentation:** Restrict management protocols (SSH, RDP, SNMP) to dedicated Management VLANs via Access Control Lists (ACLs).
