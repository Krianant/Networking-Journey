# 🛡️ Cisco Access Control Lists (ACLs) Fundamentals

An **Access Control List (ACL)** is an ordered set of sequential permit or deny rules applied to router or Layer 3 switch interfaces to filter traffic based on packet headers.

---

## ⚖️ Standard vs. Extended ACLs

| Feature | Standard ACL | Extended ACL |
| :--- | :--- | :--- |
| **Number Ranges** | `1 – 99` and `1300 – 1999` (Expanded) | `100 – 199` and `2000 – 2699` (Expanded) |
| **Named ACLs** | Supported (`ip access-list standard <NAME>`) | Supported (`ip access-list extended <NAME>`) |
| **Filtering Criteria** | **Source IPv4 Address ONLY** | **Source IP, Destination IP, Protocol (TCP/UDP/ICMP), and Port Numbers** |
| **Placement Best Practice** | **Closest to the DESTINATION** | **Closest to the SOURCE** |

> 🔑 **Placement Rules of Thumb:**
> - **Standard ACLs:** Filter on source only. If you place a Standard ACL close to the source, you risk blocking traffic to destinations you wanted to allow! Therefore, place Standard ACLs **as close to the destination as possible**.
> - **Extended ACLs:** Filter on source, destination, protocol, and port. Place them **as close to the source as possible** to prevent unwanted packets from traversing the network backbone unnecessarily.

---

## 🚫 The Implicit Deny Any

Every Cisco ACL ends with an invisible, unwritten rule:
```text
deny any (for Standard)
deny ip any any (for Extended)
```
> ⚠️ If a packet does not match any permit statement in the ACL, it is automatically **dropped and discarded**! Therefore, every functional ACL must include at least one `permit` statement.

---

## 🎭 Wildcard Masking

Cisco ACLs use **Wildcard Masks** rather than standard subnet masks:
- Binary `0` = **Match** the bit exactly.
- Binary `1` = **Ignore** the bit ("I don't care").

### Formula:
$$\text{Wildcard Mask} = 255.255.255.255 - \text{Subnet Mask}$$

### Examples:
- Subnet mask `255.255.255.0` (/24) $\rightarrow$ Wildcard: `0.0.0.255`
- Subnet mask `255.255.255.224` (/27) $\rightarrow$ Wildcard: `0.0.0.31`
- Single Host (`255.255.255.255` /32) $\rightarrow$ Wildcard: `0.0.0.0` (or keyword `host <IP>`)
- Any Host (`0.0.0.0` /0) $\rightarrow$ Wildcard: `255.255.255.255` (or keyword `any`)

---

## 📝 Configuration Examples

### 1. Standard Numbered ACL
Block host `192.168.10.5` from accessing LAN 2 (`192.168.20.0/24`), but allow all other traffic:
```cisco
! Create the ACL
Router(config)# access-list 10 deny host 192.168.10.5
Router(config)# access-list 10 permit any

! Apply to outgoing interface closest to destination
Router(config)# interface GigabitEthernet0/0/1
Router(config-if)# ip access-group 10 out
Router(config-if)# exit
```

---

### 2. Extended Named ACL (Enterprise Standard)
Requirements:
1. Allow Staff Subnet `192.168.10.0/24` to access Web Server `192.168.30.50` only via HTTP (80) and HTTPS (443).
2. Allow Staff Subnet to ping (`icmp`) the Web Server.
3. Deny all other access from Staff Subnet to the Web Server.
4. Allow all other traffic.

```cisco
! Create named extended ACL
Router(config)# ip access-list extended FILTER-STAFF-ACCESS
Router(config-ext-nacl)# remark Permit HTTP and HTTPS to Web Server
Router(config-ext-nacl)# permit tcp 192.168.10.0 0.0.0.255 host 192.168.30.50 eq 80
Router(config-ext-nacl)# permit tcp 192.168.10.0 0.0.0.255 host 192.168.30.50 eq 443

Router(config-ext-nacl)# remark Permit ICMP Echo to Web Server
Router(config-ext-nacl)# permit icmp 192.168.10.0 0.0.0.255 host 192.168.30.50 echo

Router(config-ext-nacl)# remark Deny all other access to this specific server
Router(config-ext-nacl)# deny ip 192.168.10.0 0.0.0.255 host 192.168.30.50

Router(config-ext-nacl)# remark Permit remaining outbound traffic
Router(config-ext-nacl)# permit ip 192.168.10.0 0.0.0.255 any
Router(config-ext-nacl)# exit

! Apply inbound on the ingress interface closest to source
Router(config)# interface GigabitEthernet0/0/0
Router(config-if)# ip access-group FILTER-STAFF-ACCESS in
Router(config-if)# exit
```

---

## 🛠️ Verification & Monitoring Commands

```cisco
! View all configured ACLs and match hit counters
Router# show access-lists

! Check which ACL is applied to an interface and in which direction
Router# show ip interface GigabitEthernet0/0/0

! Clear match hit counters
Router# clear access-list counters
```

Sample output of `show access-lists`:
```text
Extended IP access list FILTER-STAFF-ACCESS
    10 permit tcp 192.168.10.0 0.0.0.255 host 192.168.30.50 eq www (248 matches)
    20 permit tcp 192.168.10.0 0.0.0.255 host 192.168.30.50 eq 443 (1420 matches)
    30 permit icmp 192.168.10.0 0.0.0.255 host 192.168.30.50 echo (12 matches)
    40 deny ip 192.168.10.0 0.0.0.255 host 192.168.30.50 (3 matches)
    50 permit ip 192.168.10.0 0.0.0.255 any (8502 matches)
```
*(The hit counters allow immediate identification of matched rules and troubleshooting dropped packets!)*
