# Multi-OS Enterprise Browser Security & Policy Enforcement

## 📌 Project Overview
This project demonstrates the design, deployment, and enforcement of centralized enterprise browser safety and privacy policies across a hybrid homelab network. Using **Windows Active Directory Domain Services (AD DS)**, **Group Policy Objects (GPOs)**, and native **Linux Enterprise JSON policy mapping**, this architecture locks down browser endpoints across **Windows 11** and a **Raspberry Pi 5 (Debian Linux)**.

The core goal is to enforce immutable client-side safety nets (Strict SafeSearch, Restricted YouTube Mode, DNS-over-HTTPS blocking, and Forced Default Homepages) at the OS and user-profile level, preventing endpoints from bypassing network-level content filters like **OPNsense**.

---

## 🏗️ System Architecture
The environment targets users dynamically based on their Active Directory Organizational Unit (OU) placement and handles Windows and Linux configurations seamlessly.

```text
                  [ Windows Domain Controller ]
                                |
        +-----------------------+-----------------------+
        | (GPO Enforced)                                | (JSON Synced Fallback)
        v                                               v
[ Windows 11 Clients ]                         [ Raspberry Pi 5 Client ]
  - Google Chrome                                - Mozilla Firefox / Firefox ESR
  - Microsoft Edge                               - Machine Name: SCHOOLBOX
  - Target: Kids OU                              - Target: Kids OU Folder
