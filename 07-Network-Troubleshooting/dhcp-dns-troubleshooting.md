# 🩺 DHCP & DNS Troubleshooting Guide

Two of the most frequent network service failures encountered in helpdesk, network operations, and CCNA labs involve **DHCP (Automatic Addressing)** and **DNS (Name Resolution)**.

---

## ⚡ Part 1: DHCP Troubleshooting

DHCP (Dynamic Host Configuration Protocol) operates over **UDP Ports 67 (Server)** and **68 (Client)**.

### The DORA Process:
```text
  Client                                                      DHCP Server
    │                                                              │
    │ ── 1. DHCP Discover (Broadcast: 255.255.255.255) ──────────> │
    │       "I need an IP address!"                                │
    │                                                              │
    │ <── 2. DHCP Offer (Unicast/Broadcast) ────────────────────── │
    │       "Here is 192.168.1.50 with mask, router, & DNS"        │
    │                                                              │
    │ ── 3. DHCP Request (Broadcast) ────────────────────────────> │
    │       "I accept 192.168.1.50 from this server!"              │
    │                                                              │
    │ <── 4. DHCP Acknowledgment (Unicast/Broadcast) ───────────── │
    │       "Lease confirmed! You are good to go."                 │
```

---

### Common DHCP Failures & Root Causes

#### 1. Host Receives APIPA (`169.254.X.X`)
- **Symptom:** Client shows IP `169.254.10.45` with subnet mask `255.255.0.0`.
- **Cause:** Client sent a DHCP Discover, but received **zero** DHCP Offers before timeout.
- **Troubleshooting Steps:**
  1. Check physical link / cable.
  2. Check if the switch port is in the correct VLAN.
  3. Verify the DHCP server process is running and the pool is not exhausted.
  4. If the DHCP server is on a **different subnet/VLAN**, verify `ip helper-address` on the router!

#### 2. The IP Helper-Address (DHCP Relay) Problem
- **Concept:** Routers drop broadcast packets (`255.255.255.255`) by default. If your centralized DHCP Server (`192.168.100.10`) is located on a server subnet, DHCP Discovers from client subnets are dropped at the router interface!
- **Fix:** Configure `ip helper-address` on the **client-facing router interface**:
  ```cisco
  interface GigabitEthernet0/0/0
   description Client-LAN
   ip address 192.168.10.1 255.255.255.0
   ! Forwards broadcast DHCP Discovers as unicast packets directly to DHCP server
   ip helper-address 192.168.100.10
  ```

#### 3. Configuring Cisco IOS as a DHCP Server
```cisco
! 1. Exclude statically assigned IPs (Gateways, Printers, Servers)
ip dhcp excluded-address 192.168.10.1 192.168.10.10
ip dhcp excluded-address 192.168.10.254

! 2. Create the DHCP Pool
ip dhcp pool STAFF-POOL
 network 192.168.10.0 255.255.255.0
 default-router 192.168.10.1
 dns-server 8.8.8.8 1.1.1.1
 domain-name company.local
 lease 1 0 0    ! 1 day lease
exit
```

#### 4. DHCP Verification Commands
```cisco
! View currently leased IP addresses and client MAC addresses
Router# show ip dhcp binding

! Check pool utilization and free addresses
Router# show ip dhcp pool

! Check IP address conflicts detected by ping before assignment
Router# show ip dhcp conflict

! Clear active conflicts
Router# clear ip dhcp conflict *
```

On Client:
```cmd
:: Release current IP and request a new lease
ipconfig /release
ipconfig /renew
```

---

## 🌐 Part 2: DNS Troubleshooting

DNS (Domain Name System) operates on **UDP/TCP Port 53**.

### Typical DNS Failure Symptoms:
- Web browsing works when entering IP addresses (e.g. `http://93.184.216.34`), but fails when using domain names (`http://example.com`).
- Error messages: `DNS_PROBE_FINISHED_NXDOMAIN`, `Server Not Found`.

---

### Step-by-Step DNS Troubleshooting Checklist

```text
Step 1: Check Client DNS Configuration
   │  Windows: ipconfig /all
   │  Linux: cat /etc/resolv.conf
   ▼
Step 2: Test Ping to the Configured DNS Resolver
   │  ping 8.8.8.8 (Verify network path to the DNS server is reachable)
   ▼
Step 3: Test Direct DNS Resolution using nslookup or dig
   │  nslookup google.com 8.8.8.8
   ▼
Step 4: Check for Local DNS Cache Corruption
   │  ipconfig /flushdns (Windows)
   │  resolvectl flush-caches (Linux)
   ▼
Step 5: Check Port 53 Firewall Rules
   │  Test-NetConnection -ComputerName 8.8.8.8 -Port 53
```

---

### Hands-On Diagnostic Commands

#### 1. Windows `nslookup`
```cmd
:: Test with default configured DNS
nslookup google.com

:: Force query against an explicit DNS server (e.g., Cloudflare 1.1.1.1)
nslookup google.com 1.1.1.1

:: Reverse DNS Lookup (IP to Hostname / PTR record)
nslookup 8.8.8.8
```

#### 2. Linux `dig` (Domain Information Groper)
```bash
# Query with clean, concise output
dig google.com +short

# Query against specific resolver with full debugging output
dig @1.1.1.1 google.com

# Query specific record types
dig example.com MX       # Mail servers
dig example.com TXT      # SPF / DMARC verification
dig example.com SOA      # Start of Authority
```

#### 3. Clear and Reset Client DNS
- **Windows:**
  ```cmd
  ipconfig /flushdns
  ipconfig /registerdns
  ```
- **Linux:**
  ```bash
  sudo systemd-resolve --flush-caches
  # OR on newer Ubuntu:
  sudo resolvectl flush-caches
  ```

---

## 🎯 Summary Diagnostic Matrix

| Observation | Immediate Conclusion | Action Item |
| :--- | :--- | :--- |
| IP is `169.254.x.x` | DHCP failure (No response from server) | Check cable, VLAN, DHCP pool capacity, and `ip helper-address`. |
| `ping 8.8.8.8` works, `ping google.com` fails | Layer 3/Internet works, DNS resolution failed | Check DNS IP in `ipconfig /all`, test `nslookup`, flush cache. |
| `nslookup google.com 8.8.8.8` times out | Port 53 UDP traffic blocked | Check egress firewall or ISP DNS filtering. |
| Duplicate IP address warning on PC | Static IP configured inside dynamic DHCP range | Add static IP to `ip dhcp excluded-address` on router. |
