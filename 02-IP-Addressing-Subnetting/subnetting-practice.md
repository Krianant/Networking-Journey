# 🧮 Subnetting Guide & Practice

Subnetting is the practice of dividing a single large IP network into smaller, logically segmented subnetworks (**subnets**). 

---

## 🎯 Why Subnet?
1. **Reduce Broadcast Traffic:** Limits broadcast storms to smaller collision/broadcast domains.
2. **Improve Security:** Allows network segregation (e.g., Finance, HR, Guest Wi-Fi, Servers) via firewall policies and ACLs.
3. **Conserve IP Addresses:** Custom-tailors subnet sizes to actual host requirements using **VLSM (Variable Length Subnet Masking)**.

---

## 🔑 The "Magic Number" Method (Fast Mental Subnetting)

The **Magic Number** (or Block Size / Increment) is the key to calculating subnets quickly without doing full binary math.

### 5-Step Subnetting Formula:
1. **Identify the "Interesting Octet":** The octet where the subnet mask is neither `255` nor `0`.
2. **Calculate the Magic Number:**
   $$\text{Magic Number} = 256 - (\text{value of the interesting octet in subnet mask})$$
3. **Find the Subnet Multiples:** Multiples of the Magic Number starting at `0` ($0, M, 2M, 3M, \dots$).
4. **Determine the Network ID:** Find the multiple that is less than or equal to the host's IP in that octet.
5. **Determine the Subnet Boundaries:**
   - **Network ID:** First address ($H_{\text{all}} = 0$).
   - **First Usable Host:** $\text{Network ID} + 1$.
   - **Broadcast Address:** Next Subnet ID $- 1$.
   - **Last Usable Host:** $\text{Broadcast Address} - 1$.
   - **Total Usable Hosts:** $2^H - 2$ (where $H$ is the number of host bits).

---

## 📝 Worked Example 1: Standard Subnetting

**Problem:** Given host IP `192.168.10.75 /26`:
- Find Subnet Mask, Network ID, First Usable, Last Usable, and Broadcast Address.

**Solution:**
1. **/26 in dotted decimal:**
   - Prefix has 26 ones: `11111111.11111111.11111111.11000000`
   - Subnet Mask = `255.255.255.192`
2. **Interesting Octet:** 4th octet (`192`).
3. **Magic Number:** $256 - 192 = \mathbf{64}$.
4. **Subnet Blocks in 4th octet:**
   - Subnet 0: `192.168.10.0`
   - Subnet 1: `192.168.10.64`  <-- *75 falls here (between 64 and 127)*
   - Subnet 2: `192.168.10.128`
   - Subnet 3: `192.168.10.192`
5. **Results for 192.168.10.75 /26:**
   - **Network ID:** `192.168.10.64`
   - **First Usable:** `192.168.10.65`
   - **Last Usable:** `192.168.10.126`
   - **Broadcast Address:** `192.168.10.127`
   - **Usable Hosts:** $2^6 - 2 = 64 - 2 = \mathbf{62}$ hosts.

---

## 📐 Worked Example 2: Class B Subnetting

**Problem:** Given IP `172.16.45.10 /20`:
- Find Subnet Mask, Network ID, Usable Range, and Broadcast.

**Solution:**
1. **/20 in binary:**
   - `11111111.11111111.11110000.00000000`
   - Subnet Mask = `255.255.240.0`
2. **Interesting Octet:** 3rd octet (`240`).
3. **Magic Number:** $256 - 240 = \mathbf{16}$.
4. **Multiples of 16 in 3rd octet:** $0, 16, 32, 48, \dots$
   - 45 falls between 32 and 47!
5. **Results:**
   - **Network ID:** `172.16.32.0`
   - **First Usable:** `172.16.32.1`
   - **Last Usable:** `172.16.47.254`
   - **Broadcast Address:** `172.16.47.255`
   - **Total Usable Hosts:** $2^{12} - 2 = 4,096 - 2 = \mathbf{4,094}$ hosts.

---

## 🧩 Variable Length Subnet Masking (VLSM)

**Golden Rule of VLSM:** Always sort network requirements from **Largest to Smallest** before allocating blocks!

### Scenario:
You are assigned `192.168.1.0 /24`. Subnet for:
- **Sales:** 50 hosts
- **Engineering:** 25 hosts
- **Support:** 12 hosts
- **WAN Link (Point-to-Point):** 2 hosts

### Step-by-Step Allocation:

| Department | Hosts Needed | Formula: $2^H - 2 \ge \text{Hosts}$ | Host Bits ($H$) | Prefix Length ($32 - H$) | Block Size | Allocated Subnet Range | Broadcast |
| :--- | :---: | :---: | :---: | :---: | :---: | :--- | :--- |
| **Sales** | 50 | $2^6 - 2 = 62 \ge 50$ | 6 | **/26** | 64 | `192.168.1.0/26` (`.1` – `.62`) | `192.168.1.63` |
| **Engineering** | 25 | $2^5 - 2 = 30 \ge 25$ | 5 | **/27** | 32 | `192.168.1.64/27` (`.65` – `.94`) | `192.168.1.95` |
| **Support** | 12 | $2^4 - 2 = 14 \ge 12$ | 4 | **/28** | 16 | `192.168.1.96/28` (`.97` – `.110`) | `192.168.1.111` |
| **WAN Link** | 2 | $2^2 - 2 = 2 \ge 2$ | 2 | **/30** | 4 | `192.168.1.112/30` (`.113` – `.114`) | `192.168.1.115` |

Remaining unallocated space: `192.168.1.116` to `192.168.1.255` available for future expansion!

---

## 🏋️ Self-Practice Challenge Problems

Try solving these on paper, then check your answers:

1. **IP:** `10.50.120.45 /19`
   - *Mask:* `255.255.224.0`
   - *Magic Number:* $256 - 224 = 32$ in 3rd octet.
   - *Network ID:* `10.50.96.0`
   - *Broadcast:* `10.50.127.255`
   - *Usable Range:* `10.50.96.1` – `10.50.127.254`

2. **IP:** `192.168.5.210 /29`
   - *Mask:* `255.255.255.248`
   - *Magic Number:* $256 - 248 = 8$ in 4th octet.
   - *Network ID:* `192.168.5.208`
   - *Broadcast:* `192.168.5.215`
   - *Usable Range:* `192.168.5.209` – `192.168.5.214`
