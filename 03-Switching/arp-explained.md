# 🔍 Address Resolution Protocol (ARP) Explained

The **Address Resolution Protocol (ARP)** is defined in **RFC 826**. It bridges the gap between **Layer 3 (IP Logical Addressing)** and **Layer 2 (MAC Physical Addressing)** on a local broadcast domain.

When Host A wants to send an IP packet to Host B on the same subnet, it knows Host B's IP address, but its Network Interface Card (NIC) requires Host B's **hardware MAC address** to encapsulate the packet into an Ethernet frame. ARP solves this problem.

---

## 🔄 The ARP Request & Reply Cycle

```text
       Host A                                                      Host B
  192.168.1.10 /24                                            192.168.1.20 /24
00:11:22:33:44:AA                                           00:11:22:33:44:BB
       │                                                           │
       │ ── 1. ARP Request (Broadcast: FF:FF:FF:FF:FF:FF) ──────> │
       │    "Who has 192.168.1.20? Tell 192.168.1.10"             │
       │                                                           │
       │ <─ 2. ARP Reply (Unicast: 00:11:22:33:44:AA) ──────────── │
       │    "192.168.1.20 is at 00:11:22:33:44:BB"                 │
```

### 1. ARP Request:
- **Type:** Broadcast.
- **Destination MAC:** `FF:FF:FF:FF:FF:FF`
- **Sender IP:** `192.168.1.10`, **Sender MAC:** `00:11:22:33:44:AA`
- **Target IP:** `192.168.1.20`, **Target MAC:** `00:00:00:00:00:00` (Unknown)
- Every host on the subnet receives and processes the request. All non-matching hosts drop it; the host with target IP `192.168.1.20` generates a reply.

### 2. ARP Reply:
- **Type:** Unicast.
- **Destination MAC:** `00:11:22:33:44:AA` (Directly addressed to Host A)
- **Sender IP:** `192.168.1.20`, **Sender MAC:** `00:11:22:33:44:BB`
- Host A receives the reply and caches the entry in its **ARP Table** (ARP Cache).

---

## 💻 Inspecting and Managing the ARP Table

### On Windows / Linux:
```cmd
:: Display current ARP cache
arp -a

:: Delete all ARP entries (requires admin/root)
arp -d *

:: On modern Linux:
ip neigh show
ip neigh flush all
```

### On Cisco IOS:
```cisco
Router# show ip arp
Protocol  Address          Age (min)  Hardware Addr   Type   Interface
Internet  192.168.1.1             -   0001.968d.3101  ARPA   GigabitEthernet0/0
Internet  192.168.1.10           12   0002.164a.b801  ARPA   GigabitEthernet0/0

Router# clear arp-cache
```

---

## 🌟 Types of ARP

### 1. Standard ARP
Resolves an unknown MAC address for a known local IP address.

### 2. Gratuitous ARP (GARP)
An unsolicited ARP broadcast where a host announces its own IP and MAC address (`Sender IP == Target IP`):
- **Purposes:**
  1. **IP Conflict Detection:** If another host replies, an IP conflict exists.
  2. **Updating Neighbor Caches:** Used during NIC failover or First Hop Redundancy Protocols (HSRP/VRRP) so switches update their MAC tables immediately.

### 3. Proxy ARP
A router replies to an ARP request on behalf of a host located on a different network, making it seem like the remote host is locally present. (Generally disabled in modern secure networks).

---

## ☠️ Security Threat: ARP Spoofing / Poisoning (Man-in-the-Middle)

### How It Works:
ARP has **no built-in authentication**. A computer will accept and update its ARP table with any unsolicited ARP reply it receives.

1. Attacker sends forged ARP replies to Host A claiming: `192.168.1.1 (Gateway) is at Attacker-MAC`.
2. Attacker sends forged ARP replies to the Gateway claiming: `192.168.1.10 (Host A) is at Attacker-MAC`.
3. All traffic between Host A and the Internet passes through the attacker's machine, allowing packet sniffing and credential harvesting.

```text
[ Host A ] ──────────────► [ Attacker (MITM) ] ──────────────► [ Gateway / Internet ]
```

### Mitigation: Dynamic ARP Inspection (DAI)
On Cisco enterprise switches, **DAI** intercepts and validates all ARP requests and replies against the **DHCP Snooping binding database**:
```cisco
Switch(config)# ip dhcp snooping
Switch(config)# ip dhcp snooping vlan 10
Switch(config)# ip arp inspection vlan 10
Switch(config-if)# ip arp inspection trust    ! Applied on trusted uplink ports
```
