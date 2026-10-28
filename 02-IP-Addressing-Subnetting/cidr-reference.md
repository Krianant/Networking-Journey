# 📊 CIDR & Subnet Mask Quick Reference

**CIDR (Classless Inter-Domain Routing)** replaces the obsolete classful networking architecture (Class A, B, C) with variable-length prefix notation (e.g. `/24`).

---

## 📑 Complete IPv4 Subnet Mask Cheat Sheet

| CIDR | Subnet Mask | Total IPs ($2^H$) | Usable Hosts ($2^H - 2$) | Block Size | Wildcard Mask |
| :---: | :--- | :---: | :---: | :---: | :--- |
| **/8** | `255.0.0.0` | 16,777,216 | 16,777,214 | 1 (Octet 1) | `0.255.255.255` |
| **/9** | `255.128.0.0` | 8,388,608 | 8,388,606 | 128 (Octet 2) | `0.127.255.255` |
| **/10** | `255.192.0.0` | 4,194,304 | 4,194,302 | 64 (Octet 2) | `0.63.255.255` |
| **/11** | `255.224.0.0` | 2,097,152 | 2,097,150 | 32 (Octet 2) | `0.31.255.255` |
| **/12** | `255.240.0.0` | 1,048,576 | 1,048,574 | 16 (Octet 2) | `0.15.255.255` |
| **/13** | `255.248.0.0` | 524,288 | 524,286 | 8 (Octet 2) | `0.7.255.255` |
| **/14** | `255.252.0.0` | 262,144 | 262,142 | 4 (Octet 2) | `0.3.255.255` |
| **/15** | `255.254.0.0` | 131,072 | 131,070 | 2 (Octet 2) | `0.1.255.255` |
| **/16** | `255.255.0.0` | 65,536 | 65,534 | 1 (Octet 2) | `0.0.255.255` |
| **/17** | `255.255.128.0` | 32,768 | 32,766 | 128 (Octet 3) | `0.0.127.255` |
| **/18** | `255.255.192.0` | 16,384 | 16,382 | 64 (Octet 3) | `0.0.63.255` |
| **/19** | `255.255.224.0` | 8,192 | 8,190 | 32 (Octet 3) | `0.0.31.255` |
| **/20** | `255.255.240.0` | 4,096 | 4,094 | 16 (Octet 3) | `0.0.15.255` |
| **/21** | `255.255.248.0` | 2,048 | 2,046 | 8 (Octet 3) | `0.0.7.255` |
| **/22** | `255.255.252.0` | 1,024 | 1,022 | 4 (Octet 3) | `0.0.3.255` |
| **/23** | `255.255.254.0` | 512 | 510 | 2 (Octet 3) | `0.0.1.255` |
| **/24** | `255.255.255.0` | 256 | 254 | 1 (Octet 3) | `0.0.0.255` |
| **/25** | `255.255.255.128` | 128 | 126 | 128 (Octet 4) | `0.0.0.127` |
| **/26** | `255.255.255.192` | 64 | 62 | 64 (Octet 4) | `0.0.0.63` |
| **/27** | `255.255.255.224` | 32 | 30 | 32 (Octet 4) | `0.0.0.31` |
| **/28** | `255.255.255.240` | 16 | 14 | 16 (Octet 4) | `0.0.0.15` |
| **/29** | `255.255.255.248` | 8 | 6 | 8 (Octet 4) | `0.0.0.7` |
| **/30** | `255.255.255.252` | 4 | **2** | 4 (Octet 4) | `0.0.0.3` |
| **/31** | `255.255.255.254` | 2 | 2 (RFC 3021) | 2 (Octet 4) | `0.0.0.1` |
| **/32** | `255.255.255.255` | 1 | 1 (Host Route) | 1 (Octet 4) | `0.0.0.0` |

---

## 💡 Quick Tips for CCNA Exams & Real Labs

1. **/30 Subnet:** Historically the gold standard for point-to-point router links because it yields exactly 2 usable host addresses (1 for each router interface).
2. **/32 Subnet:** Represents a single, specific host. Commonly used for Router Loopback interfaces (`interface Loopback0`) and static host routes.
3. **Wildcard Mask Calculation:**
   $$\text{Wildcard Mask} = 255.255.255.255 - \text{Subnet Mask}$$
   - Example for `/27` (`255.255.255.224`):
     - $255 - 255 = 0$
     - $255 - 255 = 0$
     - $255 - 255 = 0$
     - $255 - 224 = 31$
     - Result: `0.0.0.31` (Used directly in Cisco OSPF network statements & ACLs).
