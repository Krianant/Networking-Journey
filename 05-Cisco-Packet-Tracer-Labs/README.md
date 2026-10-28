# 🧪 Cisco Packet Tracer Hands-On Labs

This directory contains detailed, reproducible network lab scenarios built in **Cisco Packet Tracer**.

Each lab is documented with:
- **Topology Diagram & Requirements**
- **Addressing Table**
- **Step-by-Step CLI Configuration**
- **Verification & Testing Commands**
- **Packet Tracer Simulation Mode Observations**
- **Troubleshooting Reflections & Key Takeaways**

---

## 🗂️ Lab Portfolio Index

| Lab | Name | Description | Key Protocols & Skills | Status |
| :---: | :--- | :--- | :--- | :---: |
| **01** | [**Basic LAN Configuration**](./basic-lan/) | Single switch connecting two end-hosts. Initial switch setup, MAC table verification, Packet Tracer simulation mode. | Ethernet, MAC Tables, ARP, ICMP | ✅ Ready |
| **02** | [**Subnetting & Addressing Lab**](./subnetting-lab/) | Implementing VLSM subnets across multiple router interfaces, configuring default gateways, testing subnet isolation. | IPv4, Subnetting, VLSM, Default Gateway | ✅ Ready |
| **03** | [**Inter-VLAN Routing (ROAS)**](./inter-vlan-routing/) | VLAN segmentation (VLAN 10 Sales, VLAN 20 Engineering), 802.1Q trunking, Router-on-a-Stick sub-interfaces. | 802.1Q, VLANs, Sub-interfaces, Trunking | ✅ Ready |
| **04** | [**Multi-Router Static Routing**](./static-routing/) | 3-router WAN topology, configuring static and default routes, floating static route failover testing. | Static Routes, Floating Routes, Administrative Distance | ✅ Ready |

---

## 📐 Lab Template Standard

When contributing a new lab, follow this standardized structure:

1. **Lab Overview & Objective**
2. **Network Topology ASCII / Mermaid Diagram**
3. **Addressing Table (Device, Interface, IP Address, Subnet Mask, Default Gateway)**
4. **Device Configurations (Clean Cisco IOS blocks)**
5. **Verification & Proof of Functionality (`ping`, `show` commands)**
6. **Key Lessons & Troubleshooting Log**
