# 🛠️ Network Connectivity Troubleshooting Methodology

Structured troubleshooting is what separates an amateur guesser from a professional Network Engineer or SOC Analyst. Rather than changing random settings, apply systematic, layered models to isolate root causes rapidly.

---

## 🧭 Troubleshooting Methodologies

```text
 ┌─────────────────────────┬─────────────────────────┬─────────────────────────┐
 │ Bottom-Up (OSI L1 -> L7)│ Top-Down (OSI L7 -> L1) │ Divide-and-Conquer      │
 ├─────────────────────────┼─────────────────────────┼─────────────────────────┤
 │ Starts at Physical      │ Starts at Application   │ Starts in the middle    │
 │ (cables, link lights).  │ (browser, DNS).         │ (pinging gateway at L3).│
 │ Best when new hardware  │ Best when only one app  │ Best for general        │
 │ or cables are deployed. │ is reporting errors.    │ connectivity tickets.   │
 └─────────────────────────┴─────────────────────────┴─────────────────────────┘
```

---

## 🪜 The 7-Step Systematic Connectivity Workflow

```text
Step 1: Check Local Interface & Physical Link (Layer 1 / Layer 2)
   │  Is cable plugged in? Are link lights solid green?
   ▼
Step 2: Verify Host IP Configuration (Layer 3)
   │  Does host have valid IP, subnet mask, and gateway (not 169.254.x.x)?
   ▼
Step 3: Ping the Local Loopback Address (127.0.0.1)
   │  Tests internal TCP/IP software stack and NIC drivers.
   ▼
Step 4: Ping the Local Default Gateway (Router IP)
   │  Tests communication across the local switch / VLAN to the router.
   ▼
Step 5: Ping a Remote IP Address (e.g., 8.8.8.8)
   │  Tests upstream routing, WAN links, and NAT through ISP.
   ▼
Step 6: Ping a Domain Name (e.g., google.com)
   │  Tests DNS resolution (Layer 7).
   ▼
Step 7: Verify Port & Application Layer Reachability
   │  Tests firewall rules, ACLs, and service availability (curl, Test-NetConnection).
```

---

## 🔍 Layer-by-Layer Root Cause & Diagnosis Guide

### Layer 1: Physical Layer
- **Common Symptoms:** Port LED unlit, network adapter disabled, cable unplugged or damaged.
- **Verification Commands:**
  - Linux: `ethtool eth0` (Check `Link detected: yes/no`, speed, duplex).
  - Cisco: `show interfaces status`, `show ip interface brief` (Check `administratively down` vs `down/down`).
- **Remedy:** Reseat cables, test with cable tester, replace patch cord, ensure interface is `no shutdown`.

### Layer 2: Data Link Layer
- **Common Symptoms:** Interface shows `up / line protocol down`, speed/duplex mismatch, VLAN tagging errors.
- **Verification Commands:**
  - Cisco: `show interfaces FastEthernet0/1` (Look for CRC errors, input errors, collisions).
  - Cisco: `show mac address-table`, `show interfaces trunk`.
- **Remedy:** Fix duplex mismatch (set both ends to auto-negotiate or hardcode), ensure switchport is assigned to correct VLAN.

### Layer 3: Network Layer
- **Common Symptoms:** IP conflict, host assigned APIPA (`169.254.x.x`), incorrect subnet mask, wrong default gateway, missing router route.
- **Verification Commands:**
  - Windows: `ipconfig /all`, `route print`
  - Linux: `ip addr show`, `ip route show`
  - Cisco: `show ip route`, `ping`
- **Remedy:** Correct static IP typos, renew DHCP lease (`ipconfig /renew`), add missing static route or verify dynamic routing neighbors.

### Layer 4: Transport Layer
- **Common Symptoms:** Ping succeeds, but browser cannot open website; SSH connection refused or times out.
- **Verification Commands:**
  - Windows PowerShell: `Test-NetConnection -ComputerName 192.168.1.10 -Port 443`
  - Linux: `nc -zv 192.168.1.10 443`, `ss -tulnp`
- **Remedy:** Check local firewall (Windows Defender / `iptables`), inspect router ACLs (`show access-lists`), ensure listening daemon is running on server.

### Layer 7: Application / DNS Layer
- **Common Symptoms:** Can ping `8.8.8.8` but cannot load `google.com`.
- **Verification Commands:**
  - Windows/Linux: `nslookup google.com`, `dig google.com`
- **Remedy:** Flush DNS cache (`ipconfig /flushdns`), configure reliable DNS resolvers (e.g. `1.1.1.1`, `8.8.8.8`).

---

## 📝 Diagnostic Quick-Reference Matrix

| What Works | What Fails | Most Probable Cause |
| :--- | :--- | :--- |
| Ping `127.0.0.1` | Ping local gateway | Corrupt NIC driver or disconnected cable |
| Ping local gateway | Ping `8.8.8.8` | Gateway route failure, ISP outage, or misconfigured NAT |
| Ping `8.8.8.8` | Ping `google.com` | DNS resolution failure (wrong DNS server configured) |
| Ping `192.168.1.50` | HTTP connection to `.50:80` | Web server process down, or port 80 blocked by host/network firewall |
