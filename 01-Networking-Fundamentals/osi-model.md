# 🌐 The OSI Model (Open Systems Interconnection)

The **OSI (Open Systems Interconnection)** reference model is a conceptual framework developed by the International Organization for Standardization (ISO) in 1984. It characterizes and standardizes the communication functions of a telecommunication or computing system into **7 distinct layers**.

---

## 🏗️ The 7 Layers Overview

| Layer # | Layer Name | PDU (Protocol Data Unit) | Primary Function | Key Devices & Protocols |
| :---: | :--- | :--- | :--- | :--- |
| **7** | **Application** | Data | User interface, network services access | HTTP, HTTPS, DNS, DHCP, SSH, FTP, Telnet |
| **6** | **Presentation** | Data | Data formatting, encryption, compression | SSL/TLS, JPEG, ASCII, MPEG, GIF |
| **5** | **Session** | Data | Session establishment, management, termination | NetBIOS, RPC, PPTP, Sockets |
| **4** | **Transport** | Segment (TCP) / Datagram (UDP) | End-to-end delivery, reliability, flow control | TCP, UDP, Port numbers (0–65535) |
| **3** | **Network** | Packet | Logical addressing (IP), path determination (routing) | IPv4, IPv6, ICMP, Routers, Layer 3 Switches |
| **2** | **Data Link** | Frame | Physical addressing (MAC), error detection, framing | Ethernet (802.3), Wi-Fi (802.11), Switches, NICs, ARP |
| **1** | **Physical** | Bits | Transmission of raw bitstream over physical media | Cables (Cat6, Fiber), Hubs, Repeaters, Connectors (RJ-45) |

> 💡 **Mnemonics:**
> - Top-to-Bottom (7 to 1): **A**ll **P**eople **S**eem **T**o **N**eed **D**ata **P**rocessing
> - Bottom-to-Top (1 to 7): **P**lease **D**o **N**ot **T**hrow **S**ausage **P**izza **A**way

---

## 🔍 Deep Dive: Layer by Layer

### Layer 7: Application
- **Role:** Directly interacts with software applications (e.g., browsers, email clients). It provides network services to the end-user.
- **Protocols:**
  - `HTTP / HTTPS` (Ports 80 / 443) – Web traffic
  - `DNS` (Port 53) – Domain name resolution
  - `DHCP` (Ports 67, 68) – Dynamic IP assignment
  - `SSH` (Port 22) – Secure remote management
  - `FTP / TFTP` (Ports 20, 21 / 69) – File transfer

### Layer 6: Presentation
- **Role:** Formats data so the receiving application can understand it. Responsible for:
  - **Translation:** E.g., ASCII to EBCDIC.
  - **Compression:** E.g., GZIP, JPEG, MP3 to minimize bandwidth.
  - **Encryption / Decryption:** E.g., TLS/SSL encryption for secure HTTPS.

### Layer 5: Session
- **Role:** Manages dialogues (sessions) between computers. It sets up, coordinates, and terminates connections between applications.
- **Modes of Transmission:**
  - *Simplex:* One-way communication (e.g., radio broadcast).
  - *Half-Duplex:* Two-way, but one at a time (e.g., walkie-talkie).
  - *Full-Duplex:* Simultaneous two-way communication (e.g., telephone call).

### Layer 4: Transport
- **Role:** Responsible for end-to-end transport and process-to-process communication using **Port Numbers**.
- **Key Concepts:**
  - **TCP (Transmission Control Protocol):** Connection-oriented, reliable, guaranteed delivery with ACKs, retransmission, and windowing.
  - **UDP (User Datagram Protocol):** Connectionless, unreliable (best effort), minimal overhead, low latency (ideal for VoIP, streaming, DNS).
  - **Segmentation & Reassembly:** Slicing large data blocks into smaller segments at the sender and reconstructing them at the receiver.
  - **Flow Control:** Prevents sender from overwhelming receiver (e.g., sliding window mechanism).

### Layer 3: Network
- **Role:** Handles **logical addressing** (IP addresses) and routes packets across multiple networks.
- **Key Concepts:**
  - **Path Determination:** Routers inspect destination IP addresses and consult their routing table to select the best path.
  - **Logical Addressing:** IPv4 (32-bit) and IPv6 (128-bit) addresses.
  - **Key Protocols:** IPv4, IPv6, ICMP (ping/traceroute), OSPF, EIGRP, BGP.
  - **Key Device:** **Router** (makes forwarding decisions based on destination IP).

### Layer 2: Data Link
- **Role:** Provides node-to-node data transfer across a local physical link. Detects and handles errors that may occur in the physical layer.
- **Divided into two sublayers:**
  1. **LLC (Logical Link Control - 802.2):** Communicates with the Network Layer, manages frame synchronization and flow control.
  2. **MAC (Media Access Control - 802.3 / 802.11):** Interacts with hardware; places frames on physical media using **MAC addresses** (48-bit / 6 bytes).
- **Key Device:** **Switch** (maintains a MAC address table to forward frames within the local network).
- **Trailer:** Adds Frame Check Sequence (**FCS**) containing a Cyclic Redundancy Check (**CRC**) to detect bit corruption.

### Layer 1: Physical
- **Role:** Transmits unstructured raw bitstreams (`0`s and `1`s) over physical communication media.
- **Specifications:** Voltage levels, pinouts, cable specifications, light pulses (fiber optics), radio frequencies (Wi-Fi).
- **Physical Media:** Copper (UTP/STP Cat5e/6/6a), Fiber optic (Single-mode, Multi-mode), Wireless (2.4GHz, 5GHz, 6GHz).
- **Key Devices:** Hubs, Repeaters, Cables, Transceivers (SFP/SFP+).

---

## 📦 Encapsulation & Decapsulation

When data travels down the stack at the sender, each layer wraps the payload with its own protocol header (and trailer at Layer 2). This is called **Encapsulation**.

At the destination, each layer strips off its corresponding header as data moves up the stack. This is called **Decapsulation**.

```text
[ Sender: Application ]
        │
   (Layer 7-5 Data)      [ Data ]
        │
   (Layer 4 Transport)   [ TCP/UDP Header | Data ]                      --> Segment / Datagram
        │
   (Layer 3 Network)     [ IP Header | TCP/UDP Header | Data ]          --> Packet
        │
   (Layer 2 Data Link)   [ MAC Header | IP Header | TCP Header | Data | FCS Trailer ] --> Frame
        │
   (Layer 1 Physical)    [ 01011001011010010101010101010101010101010 ] --> Bits
```

---

## 🛡️ Cybersecurity & Troubleshooting Takeaways

- **Troubleshooting Approach (Bottom-Up):**
  1. Check link lights and cable seating (Layer 1).
  2. Verify interface status, duplex mismatches, and MAC learning (Layer 2).
  3. Verify IP configuration, subnet mask, default gateway, and ping gateway (Layer 3).
  4. Verify port reachability, firewall rules, and TCP connection (Layer 4).
  5. Check DNS resolution and application server status (Layer 7).

- **Attack Vectors mapped to OSI Layers:**
  - **L1:** Physical tampering, wiretapping, cable severing.
  - **L2:** MAC flooding, ARP poisoning/spoofing, VLAN hopping.
  - **L3:** IP spoofing, ICMP flood, route poisoning.
  - **L4:** SYN flood, port scanning.
  - **L7:** SQL injection, XSS, HTTP flood, DNS amplification.
