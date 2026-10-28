# 🦈 Wireshark Packet Analysis: TCP Three-Way Handshake & Teardown

Every reliable TCP conversation (HTTP, HTTPS, SSH, FTP, SMTP) begins with the **TCP Three-Way Handshake** to synchronize sequence numbers and establish connection parameters, and terminates gracefully with a **Connection Teardown**.

---

## 🤝 Handshake Sequence Diagram

```text
 Client (192.168.1.50:49210)                    Server (93.184.216.34:80)
           │                                                │
   [SYN]   │ ── Seq = 0 (ISN_c), Len = 0 ─────────────────> │  [SYN-RECEIVED]
           │    MSS=1460, WS=256, SACK_PERM                 │
           │                                                │
 [SYN-ACK] │ <── Seq = 0 (ISN_s), Ack = 1 (ISN_c + 1) ──── │
           │    MSS=1460, WS=128, SACK_PERM                 │
           │                                                │
   [ACK]   │ ── Seq = 1, Ack = 1 (ISN_s + 1) ─────────────> │
           │                                                │
[ESTABLISHED]                                         [ESTABLISHED]
           │                                                │
           │ ════════════ HTTP Data Exchange ══════════════ │
```

---

## 🔬 Frame-by-Frame Wireshark Dissection

### Packet 1: SYN (Synchronize)
The client initiates the connection:
```text
Transmission Control Protocol, Src Port: 49210, Dst Port: 80, Seq: 0, Len: 0
    Source Port: 49210
    Destination Port: 80
    Sequence Number: 0    (relative sequence number)
    Acknowledgment Number: 0
    Flags: 0x002 (SYN)
        000. .... .... = Reserved: Set
        ...0 .... .... = Nonce: Set
        .... 0... .... = Congestion Window Reduced (CWR): Not set
        .... .0.. .... = ECN-Echo: Not set
        .... ..0. .... = Urgent: Not set
        .... ...0 .... = Acknowledgment: Not set
        .... .... 0... = Push: Not set
        .... .... .0.. = Reset: Not set
        .... .... ..1. = Syn: Set
        .... .... ...0 = Fin: Not set
    Window: 64240
    Checksum: 0x8a1b
    Options: (12 bytes)
        Maximum Segment Size (MSS): 1460 bytes
        TCP SACK Permitted Option: True
        Window Scale: 8 (multiply by 256)
```

**Key Takeaways from Packet 1:**
- Only the `SYN` flag is set (`Flags: 0x002`).
- The client selects a random 32-bit **Initial Sequence Number (ISN)**. (Wireshark displays this as `Seq: 0` for readability via the "Relative sequence numbers" preference).
- `Acknowledgment Number` is `0` because no data has been received yet.
- TCP Options negotiate **MSS (Maximum Segment Size)** and **Window Scale**.

---

### Packet 2: SYN-ACK (Synchronize + Acknowledge)
The server agrees to communicate:
```text
Transmission Control Protocol, Src Port: 80, Dst Port: 49210, Seq: 0, Ack: 1, Len: 0
    Source Port: 80
    Destination Port: 49210
    Sequence Number: 0    (relative sequence number, server's own ISN)
    Acknowledgment Number: 1    (confirms receipt of client's SYN: ISN_c + 1)
    Flags: 0x012 (SYN, ACK)
        .... ...1 .... = Acknowledgment: Set
        .... .... ..1. = Syn: Set
    Window: 29200
```

**Key Takeaways from Packet 2:**
- Both `SYN` and `ACK` flags are set (`Flags: 0x012`).
- Server sends its own independent random ISN (`Seq: 0`).
- Server acknowledges the client's SYN by adding `1` to the client's ISN (`Ack: 1`).

---

### Packet 3: ACK (Acknowledge)
The client acknowledges the server's parameters:
```text
Transmission Control Protocol, Src Port: 49210, Dst Port: 80, Seq: 1, Ack: 1, Len: 0
    Source Port: 49210
    Destination Port: 80
    Sequence Number: 1    (client's ISN + 1)
    Acknowledgment Number: 1    (confirms receipt of server's SYN: ISN_s + 1)
    Flags: 0x010 (ACK)
        .... ...1 .... = Acknowledgment: Set
```

**Key Takeaways from Packet 3:**
- Only the `ACK` flag is set (`Flags: 0x010`).
- The handshake is complete. Both sides are in **ESTABLISHED** state.
- Data transmission can now begin immediately.

---

## 🛑 Connection Teardown (Four-Way Handshake)

```text
 Client                                          Server
   │                                               │
   │ ── FIN, ACK (Seq = X, Ack = Y) ─────────────> │  [CLOSE-WAIT]
   │                                               │
   │ <── ACK (Seq = Y, Ack = X + 1) ────────────── │
   │                                               │
   │ <── FIN, ACK (Seq = Y, Ack = X + 1) ───────── │
   │                                               │
   │ ── ACK (Seq = X + 1, Ack = Y + 1) ──────────> │
   │                                               │
[TIME-WAIT]                                     [CLOSED]
```

### The RST (Reset) Flag:
If a connection is abruptly terminated due to a port being closed, a firewall dropping the session, or an invalid packet sequence, TCP sends a packet with the **RST** flag set (`Flags: 0x004`).

---

## 🎯 Wireshark Display Filters for TCP Analysis

```text
# Filter all TCP traffic on specific port (e.g., HTTP 80)
tcp.port == 80

# Show ONLY connection initiation packets (SYN without ACK)
tcp.flags.syn == 1 and tcp.flags.ack == 0

# Show SYN-ACK response packets
tcp.flags.syn == 1 and tcp.flags.ack == 1

# Show connection termination packets (FIN)
tcp.flags.fin == 1

# Show reset packets (Connections aborted or rejected)
tcp.flags.reset == 1

# Follow an entire specific TCP conversation stream (e.g. stream 0)
tcp.stream eq 0

# Find retransmissions (Packet loss / network congestion)
tcp.analysis.retransmission
```
