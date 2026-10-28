# ⚖️ TCP vs. UDP (Layer 4 Protocols)

At Layer 4 (Transport Layer) of both the OSI and TCP/IP models, **TCP (Transmission Control Protocol)** and **UDP (User Datagram Protocol)** are the two dominant protocols responsible for transporting data between network applications.

---

## 🚀 Quick Comparison

| Feature | TCP (RFC 793) | UDP (RFC 768) |
| :--- | :--- | :--- |
| **Connection Type** | **Connection-Oriented** (requires 3-way handshake) | **Connectionless** (no handshake required) |
| **Reliability** | **Guaranteed delivery** (ACKs and retransmissions) | **Best-effort** (packets may drop or arrive out of order) |
| **Ordering** | **Guaranteed sequencing** (Sequence numbers) | **No sequence tracking** (Application must reorder) |
| **Header Size** | **20 - 60 Bytes** | **8 Bytes fixed** |
| **Speed / Latency** | Slower (protocol overhead, acknowledgments) | Very fast, minimal latency |
| **Flow Control** | **Yes** (Sliding window mechanism) | **No** |
| **Congestion Control** | **Yes** (Slow start, Congestion avoidance) | **No** |
| **Transmission Type** | Unicast | Unicast, Broadcast, Multicast |
| **Common Use Cases** | Web (HTTP/HTTPS), SSH, FTP, SMTP, Database queries | Video streaming, VoIP, Gaming, DNS queries, DHCP, SNMP |

---

## 📦 Header Comparison

### 1. TCP Header Structure (Minimum 20 Bytes)

```text
 0                   1                   2                   3
 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|          Source Port          |       Destination Port        |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|                        Sequence Number                        |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|                    Acknowledgment Number                      |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|  Data |           |U|A|P|R|S|F|                               |
| Offset| Reserved  |R|C|S|S|Y|I|            Window             |
|       |           |G|K|H|T|N|N|                               |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|           Checksum            |        Urgent Pointer         |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|                    Options                    |    Padding    |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|                             data                              |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
```

**Key TCP Header Fields:**
- **Source / Destination Port (16 bits each):** Identifies sending and receiving processes.
- **Sequence Number (32 bits):** Tracks the byte position in the stream for ordered reassembly.
- **Acknowledgment Number (32 bits):** Next expected sequence number from sender.
- **Control Flags (6-9 bits):**
  - `SYN` (Synchronize): Initiates connection.
  - `ACK` (Acknowledgment): Confirms receipt of packets.
  - `FIN` (Finish): Gracefully closes connection.
  - `RST` (Reset): Abruptly terminates connection due to error.
  - `PSH` (Push): Informs receiver to process data immediately.
  - `URG` (Urgent): Indicates urgent data pointer is valid.
- **Window Size (16 bits):** Flow control buffer size the receiver can currently accept.

---

### 2. UDP Header Structure (Fixed 8 Bytes)

```text
 0                   1                   2                   3
 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|          Source Port          |       Destination Port        |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|            Length             |           Checksum            |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|                             data                              |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
```

**UDP Header Fields:**
- **Source Port (16 bits):** Sending port (optional, can be zero).
- **Destination Port (16 bits):** Receiving application port.
- **Length (16 bits):** Total length of UDP header + data in bytes (minimum 8).
- **Checksum (16 bits):** Error checking for header and data (optional in IPv4, mandatory in IPv6).

---

## 🤝 The TCP Three-Way Handshake

Before any application payload is sent over TCP, client and server negotiate parameters via a 3-way handshake:

```text
Client                                  Server
  │                                       │
  │ ──── SYN (seq=X) ───────────────────> │ (Server allocates buffer & resources)
  │                                       │
  │ <─── SYN-ACK (seq=Y, ack=X+1) ─────── │ (Server acknowledges X, sends own seq Y)
  │                                       │
  │ ──── ACK (seq=X+1, ack=Y+1) ────────> │ (Client acknowledges Y)
  │                                       │
[ Connection Established - Data Transfer Commences ]
```

---

## 🛑 TCP Connection Termination (Four-Way Handshake)

```text
Client                                  Server
  │                                       │
  │ ──── FIN (seq=A) ───────────────────> │
  │ <─── ACK (ack=A+1) ────────────────── │
  │                                       │ (Server finishes sending remaining data)
  │ <─── FIN (seq=B) ─────────────────── │
  │ ──── ACK (ack=B+1) ──────────────────> │
  │                                       │
[ Connection Closed ]
```

---

## 🛡️ Security Considerations

- **SYN Flood Attack (DoS):**
  - Attacker sends an onslaught of `SYN` packets from spoofed IPs without sending final `ACK`.
  - Server memory fills with half-open connections until legitimate clients are starved.
  - *Mitigations:* **SYN Cookies**, reducing SYN-RECEIVED timeout, firewall rate limiting.

- **UDP Amplification Attacks:**
  - Attacker spoofs target's IP address and sends small queries to public UDP servers (e.g., DNS, NTP, Memcached).
  - Servers return massive response payloads directed at the victim.
  - *Mitigations:* BCP 38 / anti-spoofing filters, rate limiting public resolvers.
