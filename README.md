# 🛡️ Active Directory Environment Project

A self-built Active Directory home lab simulating a small corporate network — a domain controller managing DHCP, DNS, and 1,000+ users, with a domain-joined Windows 11 client. Built in Oracle VirtualBox, following [Josh Madakor's AD home lab tutorial](https://www.youtube.com/watch?v=MHsI8hJmggI), with an unplanned but genuinely useful networking troubleshooting exercise along the way.

> **Note:** This is phase 1 of a larger project, so the setup below is intentionally a high-level summary rather than a full click-by-click walkthrough. For the detailed step-by-step, [Josh Madakor's video](https://www.youtube.com/watch?v=MHsI8hJmggI) covers it thoroughly — this README will grow alongside the project as later phases (an Azure VM over a site-to-site VPN, possibly additional connected VMs) get built out.

---

<img width="3000" height="1520" alt="image" src="https://github.com/user-attachments/assets/809eb47c-1976-492b-bc31-2efcfa190cbe" />


<img width="3000" height="1520" alt="image" src="https://github.com/user-attachments/assets/ab2ac2e2-3536-4da7-b8aa-9f506f6cd2c7" />


| Component | Role |
|---|---|
| **DC** (Windows Server 2025) | Domain Controller — AD DS, DHCP, DNS, RRAS/NAT |
| **Client1** (Windows 11 Virtual Machine) | Domain-joined workstation |
| **NIC 1 (NAT)** | DC's internet-facing adapter |
| **NIC 2 (Internal)** | DC's internal-facing adapter — gateway for the lab network |
| **DHCP Scope** | `172.16.0.100 – 172.16.0.200` |

---

## 🧰 Tools Used

- Oracle VirtualBox
- Windows Server 2025 (evaluation ISO)
- Windows 11 (ISO)
- PowerShell

---

## ✅ Setup

The initial setup involves a fair number of restarts — VirtualBox networking and AD DS both need a reboot to fully apply — so the steps below are grouped by stage rather than by individual restart.

**Domain Controller (Windows Server 2025)**

**1. VM & networking**
Create the Server 2025 VM, enable Guest Additions, and identify the two adapters (link-local IPv6 = internal, normal IPv4 = NAT). Rename both adapters and the VM to "Domain Controller." *Restart.*

**2. Promote to DC**
Set a static IP/DNS on the internal adapter, install AD Domain Services, and create the new domain. Set the DSRM password and complete promotion. *Restart.*

**3. Users & admin access**
In AD Users and Computers, create an OU (e.g. `ADMINS`), add a user inside it, and add that user to Domain Admins. *Restart, then sign in as admin.*

**4. Routing, NAT & DHCP**
Install RAS/NAT and configure NAT (Routing and Remote Access → point at the internet-facing adapter). Install DHCP, add a scope (`172.16.0.100–200`, DNS = internal NIC IP), set a lease time, and authorize the server.

**5. Bulk user creation**
Disable IE Enhanced Security Configuration. Grab [`1_CREATE_USERS.ps1`](https://github.com/joshmadakor1/AD_PS/blob/master/1_CREATE_USERS.ps1) and the names file from Josh Madakor's AD_PS repo, add your name to the list, set execution policy to Unrestricted, and run it to bulk-create ~1,000 AD users.

**Client (Windows 11) — reconstructed, please verify**

Create the client VM on the Internal Network adapter only, install Windows 11, confirm it gets an IP via DHCP, rename it, join it to the domain, and restart to log in with one of the generated accounts.

---

## 🐛 Troubleshooting: Client Lost Internet After a Network Move

The lab worked fine at home — until I moved to a different network and the client VM lost internet entirely, even though the DC itself stayed connected. Turned out to be four separate issues stacked on top of each other:

1. **DHCP typo** — the scope's Router option had transposed digits (`172.16.1.0` instead of `172.16.0.1`).
2. **IP forwarding silently disabled** — RRAS's console showed routing as "on," but the underlying registry value (`IPEnableRouter`) was still `0`, so the DC never actually forwarded packets between its two adapters.
3. **Wrong DNS server handed out via DHCP** — Option 006 pointed at the DC's own NAT-adapter address instead of its internal-facing IP, so the client had no reachable DNS server.
4. **Dead DNS forwarders** — the DC's forwarders still pointed at my home router's IP, unreachable from the new network, with no working fallback.

Isolating each layer — routing table, NAT translation logs, DNS reachability, then DNS resolution — one at a time was the most valuable part of the whole build. Following a tutorial end to end is one thing; understanding *why* each piece works when something breaks is another.

---

## 🚀 What's Next — Phase 2

This is phase 1 of a larger project. Planned next steps:

- Implement Group Policy Objects and test their effectiveness with recurring/scheduled tasks
- Connect an **Azure VM** to this lab over a **site-to-site VPN**
- Monitor traffic on that device with **Microsoft Sentinel** and **Microsoft Defender**
- Possibly add a **Linux VM** to the environment, to get practice looping different OS types into the same domain/network

---

## 🙏 Credits

Built by following [Josh Madakor's YouTube tutorial](https://www.youtube.com/watch?v=MHsI8hJmggI) on setting up a basic AD home lab in VirtualBox. Network diagram and troubleshooting writeup are my own.
