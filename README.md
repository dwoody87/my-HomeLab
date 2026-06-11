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

```

# Domain Controller: Active Directory Domain Services manages user identities and pushes Registry-based policies to Windows clients.

Central Store: Consolidated ADMX/ADML template engine managing Google Chrome, Microsoft Edge, and Mozilla Firefox.

Linux Integration (SCHOOLBOX): SSSD/Active Directory domain-joined Raspberry Pi 5 client configured to read local enterprise mapping files to mirror domain-level parameters perfectly.

## ⚙️ Phase 1: Active Directory Central Store Setup
To manage third-party browsers natively from standard Group Policy Management Tools, a pristine Central Store was constructed on the Domain Controller's Sysvol share:

Plaintext
\\homelab.lan\sysvol\homelab.lan\Policies\PolicyDefinitions\
├── chrome.admx
├── msedge.admx
├── firefox.admx
└── en-US/
    ├── chrome.adml
    ├── msedge.adml
    └── firefox.adml
## 🔒 Phase 2: Windows 11 Policy Architecture (Chrome & Edge)
A user-scoped GPO titled Kids Windows 11 Browser Safety was created and linked directly to the Kids OU folder.

# 1. Google Chrome Configuration
Force SafeSearch: Enabled -> Forces Google to append safe filtering variables to all web queries.

Enforce YouTube Restricted Mode: Set to Strict Restricted YouTube Mode.

Controls the mode of DNS-over-HTTPS: Set to Disable DNS-over-HTTPS. Prevents the browser from encapsulating DNS queries into encrypted HTTPS packets, forcing it to respect local OPNsense firewall routing.

Use built-in DNS client: Disabled -> Forces dependency on the host operating system's network stack.

Enforce Homepage: Configured Homepage URL and Startup Actions to strictly open https://www.education.com upon execution, disabling blank New Tab overrides.

# 2. Microsoft Edge Configuration
Force Google/Bing SafeSearch: Enabled.

Force YouTube Restricted Mode: Set to Strict.

Configure the DNS-over-HTTPS mode: Set to Off.

## 🐧 Phase 3: Cross-Platform Linux Deployment (Raspberry Pi 5)
Linux environments do not natively evaluate Windows registry keys. To achieve an identical posture on the domain-joined Raspberry Pi 5 (SCHOOLBOX), native Firefox enterprise engine mapping was deployed.

# 1. GPO Fallback Framework
A dedicated computer-scoped policy, Kids Pi 5 Firefox Restrictions, was engineered to target the computer object inside the Kids OU, blocking network parameters and establishing a custom search template mapping via GET methods to handle template limitations:

Custom Template URL: https://www.google.com/search?q={searchTerms}&safe=active

# 2. Local Enterprise JSON Implementation
The Pi 5 executes an un-bypassable system profile block by writing a hardened enterprise deployment structure directly into the core binaries directories (/usr/lib/firefox/distribution/policies.json & /usr/lib/firefox-esr/distribution/policies.json).

## 🛠️ Troubleshooting & Schema Verification
During deployment, the Firefox engine logged an error: Unknown policy:SearchEngine.

Investigation into the Mozilla Schema revealed a key discrepancy: Windows GPO templates use the singular key SearchEngine, whereas the native Linux JSON schema strictly requires the pluralized SearchEngines key. Fixing this structural syntax successfully resolved all processing errors.

Here is the finalized, fully functional /usr/lib/firefox/distribution/policies.json:

```text
JSON
{
  "policies": {
    "SearchEngines": {
      "Default": "Google",
      "PreventInstalls": true
    },
    "Preferences": {
      "browser.search.param.google.safe": "active"
    },
    "DNSOverHTTPS": {
      "Enabled": false,
      "Locked": true
    },
    "Homepage": {
      "URL": "[https://www.education.com](https://www.education.com)",
      "Locked": true,
      "StartPage": "homepage"
    },
    "ExtensionSettings": {
      "*": {
        "blocked_install_allowed_ids": [],
        "installation_mode": "blocked"
      }
    }
  }
}

```

## 🚀 Verification & Compliance Testing
Policy validation can be programmatically verified on endpoints to ensure zero-drift compliance:

Windows 11 Policy Autonomy: Running gpupdate /force pulls down user objects flawlessly. App uninstallation menus have been structurally hidden via policy settings to prevent the tampering or removing of defensive software.

Linux Environment Evaluation: Navigating to about:policies within Firefox on the Raspberry Pi 5 shows all active blocks running under the Active policies tab with zero remaining entries under the Errors tab.

The system completely greys out endpoint adjustments with an un-bypassable administrative warning: "Your browser is being managed by your organization."
