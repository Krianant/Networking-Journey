# 🦈 Wireshark Packet Analysis: ICMP (Ping & Traceroute)

**ICMP (Internet Control Message Protocol)** is a network-layer protocol (IP Protocol `1`) defined in **RFC 792**. It does not carry application payload; instead, it provides network diagnostic information, error reporting, and path verification.

---

## 🔬 ICMP Header Structure

```text
 0                   1                   2                   3
 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|     Type      |     Code      |          Checksum             |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|           Identifier          |        Sequence Number        |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|                             Data                              |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
```

---

## 📋 Common ICMP Types & Codes

| Type | Code | Name | Description |
| :---: | :---: | :--- | :--- |
| **0** | **0** | **Echo Reply** | Response to a ping request. |
| **3** | * | **Destination Unreachable** | Sent by a router or host when a destination cannot be reached. |
| | 0 | Net Unreachable | Router has no route to destination network. |
| | 1 | Host Unreachable | Target host is not answering ARP on local subnet. |
| | 3 | Port Unreachable | Receiving host has no service listening on requested port (UDP). |
| | 4 | Fragmentation Needed | Packet exceeds MTU but `DF` (Don't Fragment) bit is set. |
| **8** | **0** | **Echo Request** | Sent to test connectivity (`ping`). |
| **11** | **0** | **Time-to-Live Exceeded** | Sent by a router when a packet's `TTL` reaches `0` (Used by `traceroute`!). |

---

## 🔍 Wireshark Packet Breakdown: The Ping Command

When running: `ping 8.8.8.8`

### 1. Frame Analysis: ICMP Echo Request
```text
Frame 1: 74 bytes on wire (592 bits)
Ethernet II, Src: Intel_aa:bb:cc, Dst: Router_11:22:33
Internet Protocol Version 4, Src: 192.168.1.10, Dst: 8.8.8.8
    Protocol: ICMP (1)
    Time to Live: 128
Internet Control Message Protocol
    Type: 8 (Echo (ping) request)
    Code: 0
    Checksum: 0x4d5b [correct]
    Identifier (BE): 1 (0x0001)
    Sequence Number (BE): 1 (0x0001)
    Data (32 bytes): 6162636465666768... (Windows default abcdefghijklmnopqrstuvw...)
```

### 2. Frame Analysis: ICMP Echo Reply
```text
Frame 2: 74 bytes on wire (592 bits)
Internet Protocol Version 4, Src: 8.8.8.8, Dst: 192.168.1.10
    Protocol: ICMP (1)
    Time to Live: 117 (decremented by intermediate hops)
Internet Control Message Protocol
    Type: 0 (Echo (ping) reply)
    Code: 0
    Checksum: 0x555b [correct]
    Identifier (BE): 1 (0x0001)  <-- Matches Request Identifier
    Sequence Number (BE): 1       <-- Matches Request Sequence Number
```

---

## 🛰️ How Traceroute Uses ICMP & TTL

`traceroute` discovers intermediate routers by intentionally manipulating the IPv4 **TTL (Time To Live)** field:

1. **Hop 1:** Sends packet with `TTL = 1`.
   - The first router decrements TTL to `0`, drops the packet, and responds with `ICMP Type 11, Code 0 (Time-to-Live Exceeded in Transit)`.
   - Sender captures the router's IP and records the round-trip latency.
2. **Hop 2:** Sends packet with `TTL = 2`.
   - First router decrements to 1 and forwards.
   - Second router decrements to 0, drops, and returns `ICMP Type 11`.
3. **Repeats** until the destination host is reached!

---

## 🎯 Essential Wireshark Display Filters for ICMP

```text
# Filter only ICMP traffic
icmp

# Show only Echo Requests (Ping outbound)
icmp.type == 8

# Show only Echo Replies (Ping responses)
icmp.type == 0

# Show Destination Unreachable packets (Troubleshooting route issues)
icmp.type == 3

# Show TTL Exceeded packets (Traceroute analysis)
icmp.type == 11

# Match specific target IP with ICMP
ip.addr == 8.8.8.8 and icmp
```

---

## 🚨 Cybersecurity Context: ICMP Attacks

- **Ping of Death:** Malformed or oversized ICMP packets (>65,535 bytes) causing legacy OS buffer overflow crashes.
- **Smurf Attack:** Attacker broadcasts spoofed ICMP Echo Requests with the victim's IP as source to an entire subnet, causing all hosts to flood the victim with replies.
- **ICMP Tunneling:** Covert data exfiltration channel by embedding sensitive files inside the ICMP payload (`Data` field) through firewalls that leave ICMP uninspected.
