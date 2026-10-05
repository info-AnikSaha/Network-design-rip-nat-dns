# Multi-Service Campus Network for GreenTech Solutions Ltd.

Design and implementation of a two-building enterprise network in **Cisco Packet Tracer**, using **VLSM, RIPv2, NAT (PAT), DHCP with relay, DNS, Email (SMTP/POP3) and a Web server**.

> project for **CSE405 – Networking**, Department of CSE, East West University

![Network topology](docs/topology.png)

## Overview

GreenTech Solutions Ltd. runs a software-development campus split across two buildings joined by a WAN link:

| Building | Department | Hosts | Role |
|---|---|---|---|
| A – Head Office | HR | 30 | End users |
| A – Head Office | IT | 30 | IT staff |
| A – Head Office | Server Room | – | DHCP, DNS, Email, Web servers |
| B – Development Center | Development | 60 | Dev team |
| B – Development Center | Testing | 30 | QA team |

## Objectives

- Hierarchical IP plan using **VLSM** on the `192.168.0.0/22` block
- **RIPv2** dynamic routing between Building A and Building B routers
- Centralized **DHCP** server serving all department LANs (via `ip helper-address`)
- Internal **DNS** for `greentech.com`
- Internal **Email** server with per-user accounts
- **NAT overload (PAT)** on Router A using public IP `100.10.50.2`

## Addressing plan (VLSM)

Base network: `192.168.0.0/22`

| Priority | Subnet | Network | CIDR | Gateway | Broadcast |
|---|---|---|---|---|---|
| 1 | Development (Bldg B) | 192.168.0.0 | /26 | 192.168.0.1 | 192.168.0.63 |
| 2 | HR (Bldg A) | 192.168.0.64 | /27 | 192.168.0.65 | 192.168.0.95 |
| 3 | IT (Bldg A) | 192.168.0.96 | /27 | 192.168.0.97 | 192.168.0.127 |
| 4 | Testing (Bldg B) | 192.168.0.128 | /27 | 192.168.0.129 | 192.168.0.159 |
| 5 | Server Room (Bldg A) | 192.168.0.160 | /27 | 192.168.0.161 | 192.168.0.191 |
| 6 | WAN link (A ↔ B) | 192.168.0.192 | /30 | .193 / .194 | 192.168.0.195 |
| 7 | Public / ISP | 100.10.50.0 | /30 | 100.10.50.2 (NAT) | – |

## Devices

| Device | Model | Location | Role |
|---|---|---|---|
| Building_A router | Cisco 2811 (+ NM-2FE2W module) | Building A | Gateway, RIP, NAT, DHCP relay |
| Building_B router | Cisco 2811 | Building B | Gateway, RIP, DHCP relay |
| SW-A-HR, SW-A-IT, SW-A-Server | Cisco 2960 | Building A | Access switches |
| SW-B-Dev, SW-B-Test | Cisco 2960 | Building B | Access switches |
| Servers | Packet Tracer Server-PT | Server Room | DHCP, DNS, Email, Web |

Cabling: straight-through for LAN links, Serial DCE/DTE for the WAN link.

## Services

- **Routing:** `router rip` → `version 2` → `no auto-summary` on both routers
- **DHCP:** one server, one pool per department; routers forward client broadcasts with `ip helper-address 192.168.0.162`
- **DNS:** `www.greentech.com` and `mail.greentech.com` A-records pointing to the internal servers
- **Email:** SMTP (25) / POP3 (110) for the `greentech.com` domain, with accounts such as `hr1`, `it1`, `dev1`
- **Web:** HTTP server with a custom "GreenTech Intranet" landing page
- **NAT:** PAT with ACL 1 permitting `192.168.0.0/22`, outside interface using `100.10.50.2`

## Test results

| Test | Source → Destination | Method | Result |
|---|---|---|---|
| Inter-building routing | PC0 (HR) → PC4 (Dev) | Ping | ✅ Successful |
| DHCP assignment | All PCs → Server | Auto-config | ✅ IP, mask, gateway assigned |
| Web access | PC7 (Testing) → Web server | Browser | ✅ Page loaded |
| Email | PC2 (IT) → PC6 (Testing) | Email client | ✅ Sent and received |
| Internet access (NAT) | PC0 (HR) → 100.10.50.1 | Ping / Simulation | ✅ Source translated to 100.10.50.2 |

## Repository contents

```
.
├── GreenTech_Network.pkt     # Packet Tracer simulation file
├── docs/
│   ├── Project_Report.pdf    # Full lab report with configs and screenshots
│   └── topology.png          # Topology screenshot
└── README.md
```

## How to run

1. Install [Cisco Packet Tracer](https://www.netacad.com/courses/packet-tracer) (free with a Cisco Networking Academy account).
2. Open `GreenTech_Network.pkt`.
3. Wait for link lights to turn green, then try:
   - On any PC: **Desktop → IP Configuration → DHCP**
   - **Desktop → Command Prompt** → `ping 192.168.0.1` (cross-building)
   - **Desktop → Web Browser** → `http://www.greentech.com`
   - **Desktop → Email** → configure mail with server `mail.greentech.com` and send a message

## Notes

- The report's router-interface table and some server addresses differ slightly from the VLSM table and the topology screenshot (a documentation inconsistency). The VLSM table above matches the working `.pkt` topology; open the `.pkt` file for the authoritative addresses.
- Built for coursework/learning in a simulator, not production.

## License

Add a license of your choice (e.g. MIT) if you want others to reuse this work.

## Author
**Anik Saha** 
Department of CSE, East West University
