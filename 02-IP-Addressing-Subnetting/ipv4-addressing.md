# 🌐 IPv4 Addressing Fundamentals

An **IPv4 address** is a **32-bit logical identifier** assigned to every network interface connected to an IP network.

---

## 🧩 Structure of an IPv4 Address

An IPv4 address consists of 32 binary bits divided into **4 octets** (each 8 bits long), separated by dots:

```text
Dotted Decimal:     192    .    168    .     10    .      1
Binary Equivalent: 11000000 . 10101000 . 00001010 . 00000001
Octet Number:       Octet 1     Octet 2     Octet 3     Octet 4
```

Every IP address is divided into two parts:
1. **Network Portion (Network ID):** Identifies the specific network to which the host belongs.
2. **Host Portion (Host ID):** Uniquely identifies the specific device interface on that network.

The **Subnet Mask** dictates exactly where the network portion ends and the host portion begins:
- A binary `1` in the mask indicates a network bit.
- A binary `0` in the mask indicates a host bit.

---

## 🏷️ Legacy Classful Addressing

Originally (RFC 791), IPv4 was partitioned into five classes based on the high-order bits of the first octet:

| Class | First Octet Range | Leading Bits | Default Subnet Mask | Default CIDR | Network / Host Ratio | Purpose |
| :---: | :--- | :--- | :--- | :--- | :--- | :--- |
| **A** | `1.0.0.0` – `126.255.255.255` | `0...` | `255.0.0.0` | `/8` | 8 bits Net / 24 bits Host (16,777,214 hosts) | Very large networks |
| **B** | `128.0.0.0` – `191.255.255.255` | `10..` | `255.255.0.0` | `/16` | 16 bits Net / 16 bits Host (65,534 hosts) | Medium networks |
| **C** | `192.0.0.0` – `223.255.255.255` | `110.` | `255.255.255.0` | `/24` | 24 bits Net / 8 bits Host (254 hosts) | Small LANs |
| **D** | `224.0.0.0` – `239.255.255.255` | `1110` | N/A | N/A | N/A | Multicast groups (OSPF, RIP, etc.) |
| **E** | `240.0.0.0` – `255.255.255.254` | `1111` | N/A | N/A | N/A | Experimental / Research |

> ⚠️ **Note:** `127.0.0.0/8` is reserved for **Loopback** (e.g. `127.0.0.1` refers to localhost).

---

## 🏠 Private vs. Public IP Addresses (RFC 1918)

Because IPv4 addresses are limited (~4.29 billion), RFC 1918 designated three private address blocks that are **non-routable on the public Internet**. Internal networks use private IPs and translate them to public IPs via **NAT (Network Address Translation)**.

| Class | RFC 1918 Private Range | Total Addresses | Default Prefix |
| :---: | :--- | :--- | :--- |
| **Class A** | `10.0.0.0` – `10.255.255.255` | 16,777,216 | `10.0.0.0/8` |
| **Class B** | `172.16.0.0` – `172.31.255.255` | 1,048,576 | `172.16.0.0/12` (16 contiguous /16 blocks) |
| **Class C** | `192.168.0.0` – `192.168.255.255` | 65,536 | `192.168.0.0/16` (256 contiguous /24 blocks) |

---

## 🎯 Special IPv4 Addresses

| Address / Range | Name | Description |
| :--- | :--- | :--- |
| `0.0.0.0/0` | Default Route / Quad-Zero | Represents "any network" or unconfigured host during DHCP DORA. |
| `127.0.0.0/8` | Loopback Address | Used for host internal inter-process communication (`127.0.0.1`). |
| `169.254.0.0/16` | APIPA (Automatic Private IP Addressing) | Link-Local address auto-assigned by OS if DHCP server fails to respond. |
| `255.255.255.255` | Limited Broadcast | Sent to all hosts on the local physical network; routers do NOT forward it. |
| Network ID | First address of any subnet | All host bits set to `0`. Unassignable to hosts. |
| Directed Broadcast | Last address of any subnet | All host bits set to `1`. Sends packets to all hosts in that specific subnet. |

---

## 🧮 Understanding Binary Place Values

To master subnetting and addressing, memorize the 8 bit weights of an octet:

| Bit Position | $2^7$ | $2^6$ | $2^5$ | $2^4$ | $2^3$ | $2^2$ | $2^1$ | $2^0$ |
| :---: | :---: | :---: | :---: | :---: | :---: | :---: | :---: | :---: |
| **Decimal Value** | **128** | **64** | **32** | **16** | **8** | **4** | **2** | **1** |

**Converting 192 to binary:**
- $128 + 64 = 192$ $\rightarrow$ `11000000`

**Converting 168 to binary:**
- $128 + 0 + 32 + 0 + 8 + 0 + 0 + 0 = 168$ $\rightarrow$ `10101000`
