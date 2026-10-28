# 💻 Essential Network Troubleshooting Commands Reference

A cross-platform command reference comparing Windows, Linux, and Cisco IOS commands used in daily network administration, lab debugging, and cybersecurity incident response.

---

## 🧭 Command Cross-Platform Comparison

| Task / Purpose | Windows CMD / PowerShell | Linux CLI | Cisco IOS CLI |
| :--- | :--- | :--- | :--- |
| **View IP & Adapter Details** | `ipconfig /all` | `ip addr show` / `ifconfig` | `show ip interface brief` |
| **Inspect Routing Table** | `route print` / `Get-NetRoute` | `ip route show` | `show ip route` |
| **View ARP Cache** | `arp -a` | `ip neigh show` / `arp -n` | `show ip arp` |
| **Test Basic Connectivity** | `ping -n 4 <IP>` | `ping -c 4 <IP>` | `ping <IP>` |
| **Continuous Ping** | `ping -t <IP>` | `ping <IP>` (default continuous) | `ping <IP> repeat 1000` |
| **Trace Hop-by-Hop Path** | `tracert -d <IP>` | `traceroute -n <IP>` / `tracepath`| `traceroute <IP>` |
| **DNS Lookup & Query** | `nslookup <domain>` | `dig <domain>` / `nslookup` | N/A (or `ping <domain>`) |
| **View Listening Ports & Sockets**| `netstat -ano` | `ss -tulnp` / `netstat -tulnp` | N/A |
| **Test Specific TCP Port** | `Test-NetConnection <IP> -Port <P>` | `nc -zv <IP> <P>` / `telnet` | `telnet <IP> <P>` |
| **Flush Local DNS Cache** | `ipconfig /flushdns` | `resolvectl flush-caches` | `clear host all *` |
| **Release & Renew DHCP** | `ipconfig /release && ipconfig /renew` | `sudo dhclient -r && sudo dhclient` | `renew dhcp <interface>` |

---

## 🔍 Deep Dive: Key Commands & Useful Flags

### 1. `ping`
Tests ICMP reachability and round-trip latency.
- **Windows:**
  ```cmd
  :: Ping with 1500 byte payload to test MTU / Fragmentation
  ping 192.168.1.1 -f -l 1472
  ```
- **Linux:**
  ```bash
  # Ping 5 times with 0.2 second intervals
  ping -c 5 -i 0.2 192.168.1.1
  ```
- **Cisco IOS Extended Ping:**
  ```cisco
  Router# ping
  Protocol [ip]: <Enter>
  Target IP address: 192.168.20.1
  Repeat count [5]: 50
  Datagram size [100]: 1500
  Source address or interface: GigabitEthernet0/0/0
  ```

---

### 2. `traceroute` / `tracert`
Identifies the router hops along the network path and reveals where packet loss occurs.
- **Windows:**
  ```cmd
  :: Fast traceroute skipping slow reverse DNS lookups
  tracert -d 8.8.8.8
  ```
- **Linux:**
  ```bash
  # ICMP-based traceroute (bypasses firewalls blocking UDP traceroute)
  traceroute -I 8.8.8.8

  # TCP SYN traceroute to check if a specific web port is filtered along path
  traceroute -T -p 443 8.8.8.8
  ```

---

### 3. `netstat` & `ss` (Socket Statistics)
Inspects active TCP connections, listening ports, and associated process IDs (PIDs).
- **Windows:**
  ```cmd
  :: Show all active connections with numerical IPs and Process IDs
  netstat -ano | findstr LISTENING
  ```
- **Linux:**
  ```bash
  # Modern, high-performance replacement for netstat:
  # -t: TCP, -u: UDP, -l: Listening, -n: Numeric, -p: Process name/PID
  ss -tulnp
  ```

---

### 4. `dig` & `nslookup` (DNS Diagnostics)
Query DNS records directly from specific recursive resolvers.
- **Query specific record type using Google DNS (`8.8.8.8`):**
  ```bash
  # Query MX (Mail Exchange) record
  dig @8.8.8.8 example.com MX +short

  # Query TXT records (SPF / DKIM / DMARC verification)
  dig @8.8.8.8 example.com TXT
  ```
- **Windows `nslookup` interactive mode:**
  ```cmd
  nslookup
  > server 1.1.1.1
  > set type=A
  > example.com
  > exit
  ```

---

### 5. `Test-NetConnection` (PowerShell Swiss Army Knife)
Combines ping, traceroute, and TCP port testing in a single cmdlet:
```powershell
# Test if a web server is accepting traffic on port 443 with detailed path routing
Test-NetConnection -ComputerName 192.168.1.50 -Port 443 -InformationLevel "Detailed"
```

---

### 6. Cisco IOS Diagnostic & Debug Commands
```cisco
! Display interface packet counters, errors, and duplex
Router# show interfaces GigabitEthernet0/0/0

! Monitor live packet routing decisions (Use with extreme caution in production!)
Router# debug ip routing
Router# debug ip icmp

! Turn off all active debugging immediately!
Router# undebug all
! OR
Router# no debug all
```
