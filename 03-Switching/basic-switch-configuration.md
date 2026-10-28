# 🛠️ Cisco IOS Basic Switch Configuration

A standard baseline configuration template and reference for configuring Cisco Catalyst switches (such as the Catalyst 2960 series) from scratch using the Cisco IOS Command-Line Interface (CLI).

---

## 🧭 Cisco IOS Command Modes Hierarchy

```text
User EXEC Mode
Switch>
   │  Type: enable
   ▼
Privileged EXEC Mode (Enable Mode)
Switch#
   │  Type: configure terminal
   ▼
Global Configuration Mode
Switch(config)#
   │
   ├── Line Configuration:      Switch(config-line)#   (e.g., line console 0, line vty 0 15)
   ├── Interface Configuration: Switch(config-if)#     (e.g., interface FastEthernet 0/1)
   └── VLAN Configuration:      Switch(config-vlan)#   (e.g., vlan 10)
```

> 💡 **Navigation Tip:**
> - Type `exit` to go back one level.
> - Type `end` or press `Ctrl + Z` to immediately return to Privileged EXEC (`Switch#`).

---

## 📋 Comprehensive Baseline Switch Configuration Script

Copy and paste this template into your switch terminal or Packet Tracer CLI tab:

```cisco
! Enter Global Configuration Mode
enable
configure terminal

! 1. Hostname Configuration
hostname SW1-Access

! 2. Prevent DNS Lookup on mistyped commands
no ip domain-lookup

! 3. Secure Privileged EXEC Mode with an encrypted password
enable secret Cisco123!

! 4. Secure Console Port
line console 0
 password ConsolePass123!
 login
 logging synchronous
 exec-timeout 10 0
exit

! 5. Secure Remote Access (VTY Lines for Telnet / SSH)
line vty 0 15
 password VtyPass123!
 login
 logging synchronous
 exec-timeout 10 0
exit

! 6. Encrypt all plain text passwords in the configuration file
service password-encryption

! 7. Legal Warning Banner (MOTD)
banner motd #
****************************************************************
* WARNING: Unauthorized access to this device is strictly      *
* prohibited. All activities are monitored and logged.         *
****************************************************************
#

! 8. Configure Management IP Address (SVI - Switch Virtual Interface)
interface vlan 1
 description Management-SVI
 ip address 192.168.1.2 255.255.255.0
 no shutdown
exit

! 9. Configure Default Gateway (Allows remote subnet management)
ip default-gateway 192.168.1.1

! 10. Exit and Save Configuration
end
copy running-config startup-config
```

---

## 🔍 Line-by-Line Command Explanation

| Command | Purpose |
| :--- | :--- |
| `no ip domain-lookup` | Stops the switch from pausing for minutes trying to resolve typos to DNS broadcast. |
| `enable secret <pwd>` | Encrypts the privileged mode password using MD5 / SHA-256 (stronger than `enable password`). |
| `logging synchronous` | Keeps CLI output clean by preventing system log messages from interrupting your active typing. |
| `exec-timeout 10 0` | Automatically logs out inactive console/VTY sessions after 10 minutes and 0 seconds. |
| `service password-encryption` | Encrypts all cleartext passwords (console, VTY) in `show running-config` with Cisco Type 7 cipher. |
| `banner motd #...#` | Displays a legal warning before login banner (crucial for legal admissibility in cybersecurity). |
| `interface vlan 1` | Default Layer 3 interface on a switch used strictly for remote IP management (SSH/Ping). |
| `ip default-gateway` | Directs switch management traffic toward the router for management from remote subnets. |
| `copy run start` | Saves current RAM running configuration to non-volatile NVRAM so it persists after reboot. |

---

## 🛠️ Essential Switch Verification Commands

```cisco
! Check current running configuration
SW1-Access# show running-config

! Check saved startup configuration in NVRAM
SW1-Access# show startup-config

! Inspect status and IP of all interfaces
SW1-Access# show ip interface brief

! Inspect physical port status, speed, and duplex
SW1-Access# show interfaces status

! Check dynamic MAC address table
SW1-Access# show mac address-table

! Verify VLAN database
SW1-Access# show vlan brief

! View hardware specs, IOS version, and uptime
SW1-Access# show version
```

---

## 🔄 Factory Resetting a Switch

To erase configuration and return a Cisco switch to factory default state:
```cisco
SW1-Access# erase startup-config
Erasing the nvram filesystem will remove all configuration files! Continue? [confirm] <Enter>

SW1-Access# delete vlan.dat
Delete filename [vlan.dat]? <Enter>
Delete flash:/vlan.dat? [confirm] <Enter>

SW1-Access# reload
Proceed with reload? [confirm] <Enter>
```
