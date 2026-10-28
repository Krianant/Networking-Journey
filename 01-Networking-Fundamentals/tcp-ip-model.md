# 🌐 The TCP/IP Model

The **TCP/IP Model** (also known as the Department of Defense / DoD Model or Internet Protocol Suite) is the practical architectural foundation of the modern Internet. While the OSI model is a 7-layer theoretical standard, the TCP/IP model was designed around real protocols deployed in ARPANET and the global Internet.

---

## 📊 Comparison: OSI Model vs. TCP/IP Model

There are two common representations of the TCP/IP model:
1. **Original 4-Layer DoD Model** (Application, Transport, Internet, Network Access)
2. **Updated 5-Layer Model** (Application, Transport, Network, Data Link, Physical)

```text
  OSI 7-Layer Model           TCP/IP 5-Layer Model        TCP/IP 4-Layer (DoD)
┌───────────────────────┐   ┌───────────────────────┐   ┌───────────────────────┐
│ 7. Application        │   │                       │   │                       │
│ 6. Presentation       │───│ 5. Application        │───│ 4. Application        │
│ 5. Session            │   │                       │   │                       │
├───────────────────────┤   ├───────────────────────┤   ├───────────────────────┤
│ 4. Transport          │───│ 4. Transport          │───│ 3. Transport (Host-Host)
├───────────────────────┤   ├───────────────────────┤   ├───────────────────────┤
│ 3. Network            │───│ 3. Network (Internet) │───│ 2. Internet           │
├───────────────────────┤   ├───────────────────────┤   ├───────────────────────┤
│ 2. Data Link          │───│ 2. Data Link          │───│ 1. Network Access     │
├───────────────────────┤   ├───────────────────────┤   │    (Link Layer)       │
│ 1. Physical           │───│ 1. Physical           │───│                       │
└───────────────────────┘   └───────────────────────┘   └───────────────────────┘
```

---

## 🔍 Detailed Breakdown of TCP/IP Layers

### 1. Application Layer (OSI Layers 5, 6, 7)
- **Purpose:** Combines the functionality of OSI Application, Presentation, and Session layers.
- **Functions:**
  - Manages high-level protocols and user interface representations.
  - Handles encryption, compression, and session tracking directly within the application or library level (e.g., OpenSSL).
- **Core Protocols:**
  - Web: `HTTP` (80), `HTTPS` (443)
  - Name Resolution: `DNS` (53)
  - Address Configuration: `DHCP` (67/68)
  - Remote Access: `SSH` (22), `Telnet` (23)
  - Email: `SMTP` (25), `IMAP` (143), `POP3` (110)
  - File Transfer: `FTP` (20/21), `TFTP` (69)
  - Network Management: `SNMP` (161/162), `NTP` (123)

### 2. Transport Layer (Host-to-Host) (OSI Layer 4)
- **Purpose:** Manages process-to-process communication, connection state, multiplexing via port numbers, and reliability.
- **Protocols:**
  - **TCP (RFC 793):** Transmission Control Protocol — connection-oriented, ordered, reliable byte stream with windowing and congestion control.
  - **UDP (RFC 768):** User Datagram Protocol — connectionless, lightweight, unverified delivery for time-sensitive traffic.
  - **SCTP (Stream Control Transmission Protocol):** Message-oriented, multi-homing support (used in telecom / VoIP signaling).

### 3. Internet Layer (Network Layer) (OSI Layer 3)
- **Purpose:** Logical addressing and packet routing across heterogeneous networks.
- **Protocols:**
  - **IP (IPv4 / IPv6):** Transports packets from source IP to destination IP.
  - **ICMP (Internet Control Message Protocol):** Error reporting and diagnostics (ping, traceroute).
  - **IGMP (Internet Group Management Protocol):** Manages IP multicast group memberships.
  - **IPsec:** Suite of protocols providing network-layer encryption, integrity, and authentication.

### 4. Network Access Layer (Link Layer) (OSI Layers 1 & 2)
- **Purpose:** Handles physical network interfacing, hardware addressing (MAC), framing, and transmission of electrical/optical/radio signals over physical media.
- **Standards & Protocols:**
  - **Ethernet (IEEE 802.3):** Copper and fiber local area networks.
  - **Wi-Fi (IEEE 802.11):** Wireless local area networks.
  - **ARP (Address Resolution Protocol):** Resolves IP addresses (Layer 3) to MAC addresses (Layer 2).
  - **PPP (Point-to-Point Protocol):** Direct connection between two nodes.

---

## 🔄 Protocol Stack in Action: Web Request Flow

When you navigate to `http://example.com` in your web browser:

1. **DNS Lookup (L7 / L4 / L3 / L2):**
   - Host generates a DNS query for `example.com` (UDP Port 53).
   - If MAC of default gateway is unknown, host sends an **ARP request**.
   - Gateway replies with its MAC address.
   - DNS request is routed and answered with the IP address of `example.com`.

2. **TCP Three-Way Handshake (L4):**
   - Host sends `SYN` to destination IP on port 80.
   - Server responds with `SYN-ACK`.
   - Host responds with `ACK`. Connection is **ESTABLISHED**.

3. **HTTP GET Request & Response (L7):**
   - Host sends `GET / HTTP/1.1`.
   - Server returns `HTTP/1.1 200 OK` with HTML payload.

4. **Connection Termination (L4):**
   - Graceful closure using `FIN` and `ACK` or `RST`.

---

## 📌 Summary Reference Table

| Layer | PDU Name | Addressing Used | Example Protocols |
| :--- | :--- | :--- | :--- |
| **Application** | Data / Message | FQDN, URL, Process ID | HTTP, DNS, DHCP, SSH |
| **Transport** | Segment (TCP) / Datagram (UDP) | Port Numbers (e.g. :80, :443) | TCP, UDP |
| **Internet** | Packet | IP Addresses (e.g. 192.168.1.1) | IPv4, IPv6, ICMP, ARP |
| **Network Access** | Frame / Bits | MAC Addresses (e.g. AA:BB:CC:DD:EE:FF) | Ethernet, 802.11, Fiber |
