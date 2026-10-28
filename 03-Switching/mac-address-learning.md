# 🖧 Switch MAC Address Learning & Frame Forwarding

Ethernet switches operate primarily at **Layer 2 (Data Link Layer)** of the OSI model. Unlike hubs (which blindly broadcast every electrical signal out of all ports), switches make intelligent forwarding decisions using a dynamic **MAC Address Table** (also known as the **CAM (Content Addressable Memory) Table**).

---

## 🏷️ Structure of an Ethernet Frame & MAC Address

### MAC Address Structure (48 Bits / 6 Bytes)
A MAC address is usually displayed as 12 hexadecimal characters (e.g., `00:1A:2B:3C:4D:5E` or Cisco format `001a.2b3c.4d5e`).

```text
 ┌───────────────────────┬───────────────────────┐
 │   OUI (24 bits)       │  Vendor Assigned (24) │
 └───────────────────────┴───────────────────────┘
  Organizationally Unique        NIC Serial ID
        Identifier
  (Assigned by IEEE)
```

### Ethernet II Frame Format:
```text
┌──────────────┬──────────────┬──────────────┬──────────┬──────────────┬─────────┐
│ Preamble/SFD │ Dest MAC     │ Source MAC   │ Type/Len │ Payload/Data │   FCS   │
│   (8 bytes)  │  (6 bytes)   │  (6 bytes)   │ (2 bytes)│ (46-1500 B)  │ (4 byte)│
└──────────────┴──────────────┴──────────────┴──────────┴──────────────┴─────────┘
```

---

## ⚙️ The Switch Learning & Forwarding Logic

Whenever an Ethernet frame enters a switch port, the switch performs two independent actions:

### Step 1: Learning (Source MAC Inspection)
- The switch inspects the **Source MAC address** of the incoming frame.
- If the Source MAC is **not** in the MAC address table:
  - The switch adds a new entry mapping that MAC address to the incoming port and VLAN.
  - The switch resets the **Aging Timer** (default is 300 seconds / 5 minutes on Cisco switches).
- If the MAC is already present on that port, the aging timer is simply refreshed.

> 🔑 **Golden Rule:** Switches learn MAC addresses **ONLY** from the **Source MAC** of incoming frames!

---

### Step 2: Forwarding (Destination MAC Inspection)
- The switch inspects the **Destination MAC address** of the frame to determine where to forward it:

```text
                               Destination MAC in Frame?
                                          │
            ┌─────────────────────────────┴─────────────────────────────┐
            ▼                                                           ▼
     [ Known Unicast ]                                          [ Unknown Unicast,
  (MAC is found in table)                                        Broadcast, Multicast ]
            │                                                           │
            ▼                                                           ▼
Forward out ONLY the specific                               Flooded out ALL ports
egress port associated with MAC                             in the same VLAN, EXCEPT
(Filtered from other ports)                                 the receiving port!
```

1. **Known Unicast:** If the destination MAC is already present in the table, the frame is forwarded out **only** that specific port.
2. **Unknown Unicast Flooding:** If the destination MAC is not in the table, the switch floods the frame out all ports belonging to the same VLAN, **except the port on which it was received**.
3. **Broadcast / Multicast:**
   - Broadcast (`FF:FF:FF:FF:FF:FF`): Always flooded to all ports in that broadcast domain.
   - Multicast (`01:00:5E:...` or `33:33:...`): Flooded unless IGMP Snooping is enabled.

---

## 🔄 Step-by-Step Scenario Walkthrough

```text
 Host A (Port Fa0/1)                                 Host B (Port Fa0/2)
 IP: 192.168.1.10                                    IP: 192.168.1.20
 MAC: AAAA.AAAA.AAAA       ┌──────────────────┐      MAC: BBBB.BBBB.BBBB
         │                 │   Cisco 2960     │               │
         └─────────────────┤     Switch       ├───────────────┘
                           │ (Table: Empty)   │
                           └──────────────────┘
                                    │
                           Host C (Port Fa0/3)
                           IP: 192.168.1.30
                           MAC: CCCC.CCCC.CCCC
```

1. **Host A pings Host B (`192.168.1.20`):**
   - Host A sends an ARP request (Dest MAC = `FFFF.FFFF.FFFF`, Source MAC = `AAAA.AAAA.AAAA`).
2. **Switch Fa0/1 receives the frame:**
   - **Learn:** Maps `AAAA.AAAA.AAAA` $\rightarrow$ `Fa0/1` in its table.
   - **Forward:** Destination is Broadcast (`FFFF.FFFF.FFFF`) $\rightarrow$ Floods out `Fa0/2` and `Fa0/3`.
3. **Host C receives frame:**
   - Sees destination IP `192.168.1.20` does not match its own IP (`192.168.1.30`). Drops the frame silently.
4. **Host B receives frame:**
   - Matches its IP! Sends an ARP reply (Dest MAC = `AAAA.AAAA.AAAA`, Source MAC = `BBBB.BBBB.BBBB`).
5. **Switch Fa0/2 receives ARP reply:**
   - **Learn:** Maps `BBBB.BBBB.BBBB` $\rightarrow$ `Fa0/2`.
   - **Forward:** Looks up destination `AAAA.AAAA.AAAA` $\rightarrow$ Found on `Fa0/1`!
   - Forwards directly out `Fa0/1` only (Host C does not receive it).
6. **Resulting MAC Table:**
   ```text
   Switch# show mac address-table
             Mac Address Table
   -------------------------------------------
   Vlan    Mac Address       Type        Ports
   ----    -----------       --------    -----
      1    aaaa.aaaa.aaaa    DYNAMIC     Fa0/1
      1    bbbb.bbbb.bbbb    DYNAMIC     Fa0/2
   ```

---

## 🛠️ Cisco IOS Verification Commands

```cisco
! Display the full dynamic MAC address table
Switch# show mac address-table

! Filter by specific interface
Switch# show mac address-table interface fastEthernet 0/1

! Filter by VLAN
Switch# show mac address-table vlan 10

! Clear dynamically learned MAC addresses
Switch# clear mac address-table dynamic
```

---

## 🚨 Security Perspective: MAC Flooding Attack

- **Attack:** An attacker floods the switch with thousands of bogus source MAC addresses in a split second using tools like `macof`.
- **Consequence:** The switch CAM table fills to capacity. Unable to learn new MACs, the switch fails open and acts like a dumb **Hub**, flooding all incoming traffic out of all ports!
- **Attacker Benefit:** Allows eavesdropping on all traffic within the VLAN.
- **Mitigation:** Cisco **Port Security**:
  ```cisco
  Switch(config-if)# switchport mode access
  Switch(config-if)# switchport port-security
  Switch(config-if)# switchport port-security maximum 2
  Switch(config-if)# switchport port-security violation shutdown
  ```
