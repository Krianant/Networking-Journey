# 🔒 Network Security Fundamentals & Layer 2 Defenses

Network Security encompasses the policies, processes, and technical controls deployed to prevent unauthorized access, misuse, modification, or denial of computer networks and network-accessible resources.

---

## 🏛️ The CIA Triad in Networking

```text
               ┌────────────────────────┐
               │     CONFIDENTIALITY    │
               │ (Encryption: IPsec/TLS)│
               └───────────┬────────────┘
                           │
             ┌─────────────┴─────────────┐
             ▼                           ▼
┌────────────────────────┐   ┌────────────────────────┐
│       INTEGRITY        │   │      AVAILABILITY      │
│ (Hashing: SHA-256/HMAC)│   │(Redundancy: HSRP, STP) │
└────────────────────────┘   └────────────────────────┘
```

- **Confidentiality:** Ensures data is only readable by authorized parties (e.g., SSH instead of Telnet, HTTPS instead of HTTP, IPsec VPN tunnels).
- **Integrity:** Ensures data is not altered in transit (e.g., FCS/CRC checks, SHA-256 digital signatures, TCP sequence numbers).
- **Availability:** Ensures network resources remain accessible when needed (e.g., redundant links, STP loop prevention, DDoS protection).

---

## ⚠️ Layer 2 Attacks & Enterprise Cisco Mitigations

Layer 2 is traditionally the soft underbelly of the network. If an attacker gains access to a physical access port or local Wi-Fi, traditional firewalls cannot protect internal hosts. Here are the most critical Layer 2 attacks and their Cisco IOS countermeasures:

### 1. MAC Address Table Overflow (MAC Flooding)
- **Mechanism:** Attacker uses tools like `macof` to flood a switch port with bogus MAC addresses, exhausting CAM table memory. The switch begins broadcasting all incoming unicast traffic out all ports.
- **Defense: Cisco Port Security**
  ```cisco
  interface FastEthernet0/5
   switchport mode access
   switchport port-security
   switchport port-security maximum 2
   switchport port-security mac-address sticky
   switchport port-security violation shutdown
  ```
  - *Violation Modes:*
    - `protect`: Drops packets from unauthorized MACs silently.
    - `restrict`: Drops packets and generates a syslog message / SNMP trap.
    - `shutdown`: Places the port into `err-disabled` state immediately.

---

### 2. DHCP Starvation & Rogue DHCP Servers
- **DHCP Starvation:** Attacker sends thousands of DHCP discover messages with spoofed MACs to deplete the IP address pool.
- **Rogue DHCP Server:** Attacker brings up a malicious DHCP server offering invalid DNS or assigning their own machine as the default gateway (Man-in-the-Middle).
- **Defense: DHCP Snooping**
  - Designates switch ports as either **Trusted** (connected to legitimate DHCP servers or uplinks) or **Untrusted** (end-user access ports).
  - Drops DHCP server reply messages (`DHCP Offer`, `DHCP Ack`) received on untrusted ports!
  ```cisco
  ip dhcp snooping
  ip dhcp snooping vlan 10,20

  ! Trust the uplink port connected to the router/DHCP server
  interface GigabitEthernet0/1
   ip dhcp snooping trust
  ```

---

### 3. ARP Spoofing / Poisoning (MITM)
- **Mechanism:** Attacker sends gratuitous ARP replies associating the gateway's IP with the attacker's MAC address, intercepting or modifying local subnet traffic.
- **Defense: Dynamic ARP Inspection (DAI)**
  - Requires DHCP Snooping to be enabled.
  - Intercepts all ARP requests and replies on untrusted ports and validates IP-to-MAC bindings against the **DHCP Snooping database**. Invalid mappings are dropped.
  ```cisco
  ip arp inspection vlan 10,20

  interface GigabitEthernet0/1
   ip arp inspection trust
  ```

---

### 4. VLAN Hopping Attacks
- **Switch Spoofing:** An attacker machine connects to a switch and negotiates an 802.1Q trunk link using **Dynamic Trunking Protocol (DTP)** to access all VLANs.
- **Double Tagging:** Attacker sends a frame with two 802.1Q tags matching the native VLAN, bypassing switch tagging logic.
- **Defense:**
  1. Disable DTP negotiation on all access ports: `switchport mode access` and `switchport nonegotiate`.
  2. Change default native VLAN (VLAN 1) on all trunks to an unused VLAN ID (e.g., VLAN 999).
  3. Shut down all unused switch ports and place them into an isolated "blackhole" VLAN.
  ```cisco
  interface range FastEthernet0/10 - 24
   switchport mode access
   switchport access vlan 999
   shutdown
  ```

---

## 📋 Network Hardening Baseline Checklist

- [ ] Disable Telnet (`no transport input telnet`) and enforce SSHv2 only.
- [ ] Implement strong passwords with `enable secret` and `service password-encryption`.
- [ ] Configure `exec-timeout` and `logging synchronous` on all console and VTY lines.
- [ ] Set up legal warning banners (`banner motd`).
- [ ] Disable unused switch ports and assign to a dummy VLAN.
- [ ] Enable Port Security on edge user ports.
- [ ] Enable DHCP Snooping and Dynamic ARP Inspection (DAI).
- [ ] Implement Control Plane Policing (CoPP) on core routers.
- [ ] Maintain synchronized system clocks across all infrastructure using NTP.
