# 🌐 Homelab Infrastructure Specification
*Version: 26.1.1*

## 📌 Project Overview
This project defines the core infrastructure and network security architecture for a centralized, domain-joined homelab environment. The architecture is designed to support virtualization, centralized storage, and identity management within a flat-network topology, bridge-connected via a perimeter OPNsense firewall.

---

## 🏗️ System Architecture
The environment utilizes a perimeter-based firewall model to separate the internal laboratory (`10.0.0.0/20`) from the home management network (`192.168.0.0/24`).

```text
[ Home Router ] (192.168.0.1)
      |
[ OPNsense Firewall ] (WAN: 192.168.0.254 | LAN: 10.0.0.1)
      |
[ Netgear GS748T ] (192.168.0.250)
      |
+-----+---------------------------+
|                                 |
[ AD/DNS Server ] (10.0.0.10)   [ VM/NFS Server ] (10.0.0.20)
(homelab.lan)                   (Hypervisor Host)
```

## 📋 Core Infrastructure Components

# 1. Network & Perimeter Security
>Perimeter Firewall: OPNsense 26.1.1 manages all inter-network traffic.
>Segment Strategy: Flat network topology (10.0.0.0/20) serving all internal virtualized workloads.
>Management Access: Administrative workstation (192.168.0.87) is explicitly permitted by firewall rule to access the 10.0.0.0/20 subnet.
>Printing: Cross-segment printing facilitated by a restricted firewall permit from 10.0.0.0/20 to the Epson ET-2980 (192.168.0.50).

# 2. Infrastructure Services
>Identity & Resolution: 10.0.0.10 (Active Directory Domain Services / DNS) serves as the authority for homelab.lan.
>Compute & Storage: 10.0.0.20 provides NFS storage and virtualization resources for all hosted workloads.

## 🖥️ Virtual Machine Inventory
All VMs are provisioned within the 10.0.0.0/20 subnet, utilizing the central AD/DNS server for authentication.

```text
   Role       |    Static IP   |              Purpose
Windows VM1   |    10.0.1.10   |     General Purpose Workstation
Windows VM2   |    10.0.2.10   |     General Purpose Workstation
Windows VM3   |    10.0.3.10   |     General Purpose Workstation
Linux VM      |    10.0.4.10   |       Server/Container Host
```

## 🔒 Firewall Rule Logic (Summary)

>WAN Interface:
  >Permit: 192.168.0.87 (Laptop) -> 10.0.0.0/20 (Lab Subnet)\
  >Port Forward: 25565 (Minecraft TCP/UDP) -> 10.0.0.25 (Game VM)

>LAN Interface:
  >Permit: 10.0.0.0/20 (Lab Subnet) -> 192.168.0.50 (Epson Printer)
  >Default: Allow internal-to-internal traffic (Flat topology).

>This configuration serves as the baseline for the Windows-Server-Deployment branch. All future deployments must verify connectivity to 10.0.0.10 (DNS/AD) prior to domain join operations.
