# 🏢 Multi-Site Enterprise Network — Cairo HQ & Alexandria Branch

![Cisco Packet Tracer](https://img.shields.io/badge/Tool-Cisco%20Packet%20Tracer-blue?logo=cisco)
![HSRP v2](https://img.shields.io/badge/Protocol-HSRP%20v2-green)
![OSPF](https://img.shields.io/badge/Protocol-OSPF%20Area%200-brightgreen)
![LACP](https://img.shields.io/badge/Protocol-LACP%20EtherChannel-orange)
![Network Design](https://img.shields.io/badge/Focus-Enterprise%20Network%20Design-red)
![Layer 3 Switching](https://img.shields.io/badge/Layer-L3%20Switching-purple)
![High Availability](https://img.shields.io/badge/Feature-High%20Availability-success)
![Status](https://img.shields.io/badge/Status-Completed-success)

> **Note on this revision:** This README was corrected to match the actual `.pkt` file exported for this lab (13 devices). A few things in the original draft don't exist in real Cisco Packet Tracer and were fixed: 3560-24PS switches only have **2** built-in GigE ports (`Gi0/1`, `Gi0/2` — not `G1/0/1…24`), router↔router links **must be fiber** (not copper), and standby interface-tracking / a second Alexandria management VLAN were removed because they were never implemented in the working config. See **"Changes from the original design"** at the end.

---

## 📑 Table of Contents

1.  [📘 Project Overview](#-project-overview)
2.  [🎯 Project Objectives](#-project-objectives)
3.  [🌐 Network Topology](#-network-topology)
4.  [🔗 Device Interface Table](#-device-interface-table)
5.  [📝 IP Addressing Table](#-ip-addressing-table)
6.  [🔧 Implementation Steps](#-implementation-steps)
7.  [💻 Device Configuration](#-device-configuration)
8.  [🛡️ Layer 2 Security Configuration](#-layer-2-security-configuration)
9.  [🌐 NAT/PAT Configuration](#-natpat-configuration)
10. [✅ Verification & Testing Scenarios](#-verification--testing-scenarios)
11. [⚡ How to Run Lab](#-how-to-run-lab)
12. [🧱 Lab Limitations](#-lab-limitations)
13. [🎓 Learning Outcomes](#-learning-outcomes)
14. [💡 Repository Info](#-repository-info)
15. [🔄 Changes from the Original Design](#-changes-from-the-original-design)

---

## 📘 Project Overview

This lab demonstrates a **multi-site enterprise network** connecting two branches (**Cairo HQ** and **Alexandria Branch**) with **full redundancy**, **centralized services**, and **end-to-end security**, exported as a working Cisco Packet Tracer `.pkt` file (13 devices, pre-configured).

The design integrates:
- **HSRP v2** for gateway redundancy at both sites
- **LACP EtherChannel** for link aggregation between core switches
- **OSPF Area 0** for primary WAN routing with a **Floating Static Route** for the backup path
- **Centralized DHCP** via DHCP Relay across the WAN
- **Layer 2 Security Hardening** (BPDU Guard, Port Security, DHCP Snooping)
- **NAT/PAT** for controlled Internet access (via a simulated ISP router)
- **Rapid PVST+** for a loop-free Layer 2 topology

---

## 🎯 Project Objectives

1.  Design a **3-Tier Architecture** (Core / Access / Edge) for Cairo HQ and a **dual-gateway** model for Alexandria Branch.
2.  Deploy **HSRP v2** between CAIRO-CORE-01/02 and ALEX-GW-01/02 for gateway redundancy.
3.  Implement **LACP EtherChannel** (Port-channel 1) between core switches for link aggregation.
4.  Configure **OSPF Area 0** on the primary WAN link with a **Floating Static Route** (AD 210) as backup.
5.  Centralize **DHCP services** in Cairo and distribute addresses to both sites via **DHCP Relay** (`ip helper-address`).
6.  Apply **Layer 2 security hardening**: BPDU Guard, Port Security (sticky MAC), DHCP Snooping.
7.  Enable **NAT/PAT** on the edge router for Internet access for both branches, verified against a simulated ISP router.
8.  Validate failover behavior across **gateway**, **WAN link**, and **port security** scenarios.

---

## 🌐 Network Topology

- **Cairo HQ**
  - **Core Layer:** `CAIRO-CORE-01` (Active) & `CAIRO-CORE-02` (Standby) — 3560-24PS L3 switches, connected via **LACP EtherChannel** on their two built-in GigE ports (`Gi0/1` + `Gi0/2`)
  - **Access Layer:** `CAIRO-ACCESS-SW01` (2960-24TT) — dual-homed trunk to both core switches, with Port Security + BPDU Guard + DHCP Snooping
  - **Edge:** `CAIRO-EDGE-GW01` (2911) — routed link to Core, OSPF + Floating Static + NAT/PAT, plus a simulated ISP router
  - **Internet simulation:** `ISP-RTR` (2911) — carries a Loopback0 `8.8.8.8/32` used as the internet ping/traceroute target
- **Alexandria Branch**
  - **Branch Gateways:** `ALEX-GW-01` (Active) & `ALEX-GW-02` (Standby) — Router-on-a-Stick with HSRP v2, each with its own dedicated fiber WAN uplink to Cairo
  - **Access:** `ALEX-SW01` (2960-24TT) — dual-homed trunk to both branch gateways
- **WAN Links (fiber, router↔router):**
  - **Primary:** `10.100.1.0/30` — OSPF Area 0 (`CAIRO-EDGE-GW01 Gi0/0/0` ↔ `ALEX-GW-01 Gi0/0/0`)
  - **Backup:** `10.100.2.0/30` — Floating Static Route AD 210 (`CAIRO-EDGE-GW01 Gi0/1/0` ↔ `ALEX-GW-02 Gi0/0/0`)
  - **Internet:** `203.0.113.0/30` — NAT outside (`CAIRO-EDGE-GW01 Gi0/2/0` ↔ `ISP-RTR Gi0/0/0`)
- **Edge ↔ Core link:** `192.168.100.0/30` — routed link (`CAIRO-EDGE-GW01 Gi0/0` ↔ `CAIRO-CORE-01 Fa0/2`, VLAN 100)
- **Centralized Services:** DHCP/DNS Server at `192.168.30.10` (VLAN 30, Cairo)

---

## 🔗 Device Interface Table

| Device | Interface | Connects To | Description |
| :--- | :--- | :--- | :--- |
| **CAIRO-CORE-01** | Gi0/1, Gi0/2 | CAIRO-CORE-02 Gi0/1, Gi0/2 | LACP EtherChannel (Port-channel 1) |
| **CAIRO-CORE-01** | Fa0/1 | CAIRO-ACCESS-SW01 Gi0/1 | 802.1Q Trunk |
| **CAIRO-CORE-01** | Fa0/2 | CAIRO-EDGE-GW01 Gi0/0 | Routed access link, VLAN 100 (`192.168.100.0/30`) |
| **CAIRO-CORE-02** | Gi0/1, Gi0/2 | CAIRO-CORE-01 Gi0/1, Gi0/2 | LACP EtherChannel (Port-channel 1) |
| **CAIRO-CORE-02** | Fa0/1 | CAIRO-ACCESS-SW01 Gi0/2 | 802.1Q Trunk |
| **CAIRO-ACCESS-SW01** | Gi0/1, Gi0/2 | CAIRO-CORE-01 / -02 Fa0/1 | Trunk uplinks, DHCP Snooping trust |
| **CAIRO-ACCESS-SW01** | Fa0/1, Fa0/2 | CAIRO-PC1, CAIRO-PC2 | Access ports, VLAN 10, Port Security |
| **CAIRO-ACCESS-SW01** | Fa0/3 | CAIRO-DHCP-DNS | Access port, VLAN 30 |
| **CAIRO-EDGE-GW01** | Gi0/0 | CAIRO-CORE-01 Fa0/2 | HQ internal routed link |
| **CAIRO-EDGE-GW01** | Gi0/0/0 (fiber) | ALEX-GW-01 Gi0/0/0 | Primary WAN — OSPF Area 0 |
| **CAIRO-EDGE-GW01** | Gi0/1/0 (fiber) | ALEX-GW-02 Gi0/0/0 | Backup WAN — Floating Static (AD 210) |
| **CAIRO-EDGE-GW01** | Gi0/2/0 (fiber) | ISP-RTR Gi0/0/0 | Internet — NAT outside |
| **ISP-RTR** | Gi0/0/0 (fiber) | CAIRO-EDGE-GW01 Gi0/2/0 | ISP link |
| **ISP-RTR** | Loopback0 | — | `8.8.8.8/32` simulated internet host |
| **ALEX-GW-01** | Gi0/0 | ALEX-SW01 Gi0/1 | Router-on-a-Stick trunk (VLAN 110, 120) |
| **ALEX-GW-01** | Gi0/0/0 (fiber) | CAIRO-EDGE-GW01 Gi0/0/0 | Primary WAN Link |
| **ALEX-GW-02** | Gi0/0 | ALEX-SW01 Gi0/2 | Router-on-a-Stick trunk (VLAN 110, 120) |
| **ALEX-GW-02** | Gi0/0/0 (fiber) | CAIRO-EDGE-GW01 Gi0/1/0 | Backup WAN Link |
| **ALEX-SW01** | Gi0/1, Gi0/2 | ALEX-GW-01 / -02 Gi0/0 | Trunk uplinks |
| **ALEX-SW01** | Fa0/1, Fa0/2 | ALEX-PC1, ALEX-PC2 | Access ports, VLAN 110 |

---

## 📝 IP Addressing Table

### Cairo HQ — 192.168.0.0/16

| Network / Component | Subnet / Address | Gateway / Purpose |
| :--- | :--- | :--- |
| **Data Network** | 192.168.10.0/24 | HSRP Virtual IP: 192.168.10.1 |
| **Voice Network** | 192.168.20.0/24 | HSRP Virtual IP: 192.168.20.1 |
| **Servers Network** | 192.168.30.0/24 | HSRP Virtual IP: 192.168.30.1 |
| **Management Network** | 192.168.99.0/24 | HSRP Virtual IP: 192.168.99.1 (Cairo core only) |
| **DHCP/DNS Server** | 192.168.30.10 | Centralized DHCP + DNS Service |
| **Edge ↔ Core Link** | 192.168.100.0/30 | Routed link, OSPF Area 0 |
| **Primary WAN Link** | 10.100.1.0/30 | OSPF Area 0 |
| **Backup WAN Link** | 10.100.2.0/30 | Floating Static (AD 210) |
| **ISP Link** | 203.0.113.0/30 | NAT Outside / Internet |
| **Simulated Internet Host** | 8.8.8.8/32 | Loopback0 on ISP-RTR |

### Alexandria Branch — 172.16.0.0/16

| Network / Component | Subnet / Address | Gateway / Purpose |
| :--- | :--- | :--- |
| **Alex Data Network** | 172.16.10.0/24 | HSRP Virtual IP: 172.16.10.1 |
| **Alex Voice Network** | 172.16.20.0/24 | HSRP Virtual IP: 172.16.20.1 |

> There is **no** separate `172.16.99.0/24` Alexandria management VLAN in the working config — it was in the original draft but never configured, so it was removed to avoid a mismatch between docs and the actual `.pkt`.

### VLAN Summary

| VLAN ID | Name | Network | Mask | Virtual Gateway | Site | Purpose |
| :-: | :--- | :--- | :--- | :--- | :--- | :--- |
| **10** | DATA_HQ | 192.168.10.0 | 255.255.255.0 | 192.168.10.1 | Cairo | End-user Data |
| **20** | VOICE_HQ | 192.168.20.0 | 255.255.255.0 | 192.168.20.1 | Cairo | VoIP (reserved, no phones in this lab) |
| **30** | SERVERS_HQ | 192.168.30.0 | 255.255.255.0 | 192.168.30.1 | Cairo | DHCP/DNS Server |
| **99** | MGMT_HQ | 192.168.99.0 | 255.255.255.0 | 192.168.99.1 | Cairo | Network Management |
| **100** | EDGE_LINK | 192.168.100.0 | 255.255.255.252 | 192.168.100.1 | Cairo | Edge↔Core routed link |
| **110** | ALEX_DATA | 172.16.10.0 | 255.255.255.0 | 172.16.10.1 | Alexandria | Branch Data |
| **120** | ALEX_VOICE | 172.16.20.0 | 255.255.255.0 | 172.16.20.1 | Alexandria | Branch VoIP (reserved) |

### HSRP Configuration Summary

| VLAN / Group | Virtual IP | Active Device | Active Priority | Standby Device | Standby Priority |
| :-: | :--- | :--- | :-: | :--- | :-: |
| 10 | 192.168.10.1 | CAIRO-CORE-01 | 110 | CAIRO-CORE-02 | 100 |
| 20 | 192.168.20.1 | CAIRO-CORE-01 | 110 | CAIRO-CORE-02 | 100 |
| 30 | 192.168.30.1 | CAIRO-CORE-01 | 110 | CAIRO-CORE-02 | 100 |
| 99 | 192.168.99.1 | CAIRO-CORE-01 | 110 | CAIRO-CORE-02 | 100 |
| 110 | 172.16.10.1 | ALEX-GW-01 | 110 | ALEX-GW-02 | 100 |
| 120 | 172.16.20.1 | ALEX-GW-01 | 110 | ALEX-GW-02 | 100 |

> `standby track` interface tracking (decrementing priority on uplink failure) is **not configured** in the working file — it was in the original draft's tables but not implemented, since it required tracking a specific uplink port that doesn't exist in this design. HSRP failover here is triggered by shutting the SVI/sub-interface itself, per the test steps below.

---

## 🔧 Implementation Steps

1.  **Core Layer Redundancy (Cairo HQ).** Created LACP EtherChannel (Port-channel 1) between `CAIRO-CORE-01` and `CAIRO-CORE-02` on their two built-in GigE ports (`Gi0/1`, `Gi0/2`). Configured HSRP v2 on all SVIs (VLAN 10/20/30/99) with `CAIRO-CORE-01` Active (priority 110) and `CAIRO-CORE-02` Standby (priority 100). Set matching Rapid-PVST+ priorities (4096 / 8192).
2.  **Branch Gateway Redundancy (Alexandria).** Configured `ALEX-GW-01`/`02` with Router-on-a-Stick sub-interfaces for VLAN 110 (Data) and VLAN 120 (Voice), each with HSRP v2 — `ALEX-GW-01` Active (110), `ALEX-GW-02` Standby (100).
3.  **HQ Internal Routing.** Connected `CAIRO-EDGE-GW01` to `CAIRO-CORE-01` over a dedicated routed VLAN (100, `192.168.100.0/30`) and ran OSPF Area 0 between them.
4.  **WAN Connectivity (OSPF + Floating Static).** Configured OSPF Area 0 on `CAIRO-EDGE-GW01` and `ALEX-GW-01` over the **primary** WAN link (`10.100.1.0/30`, fiber). Added a Floating Static Route (AD 210) on `CAIRO-EDGE-GW01` (→ `172.16.0.0/16` via `10.100.2.2`) and on `ALEX-GW-02` (→ `192.168.0.0/16` via `10.100.2.1`) over the **backup** link (`10.100.2.0/30`, fiber) — the backup link deliberately stays **out** of OSPF so the floating static only activates when the primary path is down.
5.  **Centralized DHCP Service.** Placed the DHCP/DNS server at `192.168.30.10` in Cairo (VLAN 30). Configured `ip helper-address 192.168.30.10` on the Cairo VLAN 10/20 SVIs and the Alexandria VLAN 110/120 sub-interfaces to relay DHCP requests across the WAN.
6.  **Layer 2 Security Hardening.** Enabled Rapid-PVST+, PortFast, and BPDU Guard on both access switches. Configured DHCP Snooping and Port Security (sticky MAC, max 2, violation restrict) on Cairo's user-facing ports.
7.  **Internet Access (NAT/PAT).** Defined NAT inside interfaces (`Gi0/0`, `Gi0/0/0`, `Gi0/1/0`) and NAT outside (`Gi0/2/0`) on `CAIRO-EDGE-GW01`. Applied PAT overload using ACL 100 for both Cairo (`192.168.0.0/16`) and Alexandria (`172.16.0.0/16`) subnets. Added a default route toward a simulated ISP router carrying `8.8.8.8/32` as the test target.

---

## 💻 Device Configuration

### 🏗️ CAIRO-CORE-01 (Active Core Switch)

```text
hostname CAIRO-CORE-01
ip routing
vlan 10
 name DATA_HQ
vlan 20
 name VOICE_HQ
vlan 30
 name SERVERS_HQ
vlan 99
 name MGMT_HQ
vlan 100
 name EDGE_LINK
exit
interface range GigabitEthernet0/1-2
 switchport trunk encapsulation dot1q
 switchport mode trunk
 channel-group 1 mode active
exit
interface Port-channel1
 switchport trunk encapsulation dot1q
 switchport mode trunk
exit
interface FastEthernet0/1
 switchport trunk encapsulation dot1q
 switchport mode trunk
exit
interface FastEthernet0/2
 switchport mode access
 switchport access vlan 100
exit
interface Vlan10
 description Data VLAN Gateway
 ip address 192.168.10.2 255.255.255.0
 ip helper-address 192.168.30.10
 standby version 2
 standby 10 ip 192.168.10.1
 standby 10 priority 110
 standby 10 preempt
exit
interface Vlan20
 ip address 192.168.20.2 255.255.255.0
 ip helper-address 192.168.30.10
 standby version 2
 standby 20 ip 192.168.20.1
 standby 20 priority 110
 standby 20 preempt
exit
interface Vlan30
 ip address 192.168.30.2 255.255.255.0
 standby version 2
 standby 30 ip 192.168.30.1
 standby 30 priority 110
 standby 30 preempt
exit
interface Vlan99
 ip address 192.168.99.2 255.255.255.0
 standby version 2
 standby 99 ip 192.168.99.1
 standby 99 priority 110
 standby 99 preempt
exit
interface Vlan100
 ip address 192.168.100.1 255.255.255.252
exit
router ospf 1
 router-id 1.1.1.1
 network 192.168.10.0 0.0.0.255 area 0
 network 192.168.20.0 0.0.0.255 area 0
 network 192.168.30.0 0.0.0.255 area 0
 network 192.168.100.0 0.0.0.3 area 0
exit
spanning-tree mode rapid-pvst
spanning-tree vlan 10,20,30,99 priority 4096
end
```

### 🏗️ CAIRO-CORE-02 (Standby Core Switch)

```text
hostname CAIRO-CORE-02
ip routing
vlan 10
 name DATA_HQ
vlan 20
 name VOICE_HQ
vlan 30
 name SERVERS_HQ
vlan 99
 name MGMT_HQ
exit
interface range GigabitEthernet0/1-2
 switchport trunk encapsulation dot1q
 switchport mode trunk
 channel-group 1 mode active
exit
interface Port-channel1
 switchport trunk encapsulation dot1q
 switchport mode trunk
exit
interface FastEthernet0/1
 switchport trunk encapsulation dot1q
 switchport mode trunk
exit
interface Vlan10
 ip address 192.168.10.3 255.255.255.0
 ip helper-address 192.168.30.10
 standby version 2
 standby 10 ip 192.168.10.1
 standby 10 priority 100
 standby 10 preempt
exit
interface Vlan20
 ip address 192.168.20.3 255.255.255.0
 ip helper-address 192.168.30.10
 standby version 2
 standby 20 ip 192.168.20.1
 standby 20 priority 100
 standby 20 preempt
exit
interface Vlan30
 ip address 192.168.30.3 255.255.255.0
 standby version 2
 standby 30 ip 192.168.30.1
 standby 30 priority 100
 standby 30 preempt
exit
interface Vlan99
 ip address 192.168.99.3 255.255.255.0
 standby version 2
 standby 99 ip 192.168.99.1
 standby 99 priority 100
 standby 99 preempt
exit
spanning-tree mode rapid-pvst
spanning-tree vlan 10,20,30,99 priority 8192
end
```

> **Note:** `CAIRO-CORE-02` intentionally does **not** run OSPF. Only the HSRP Active switch advertises the HQ subnets into OSPF — otherwise Packet Tracer can create duplicate/asymmetric paths. This means that if `CAIRO-CORE-01` goes down completely, HSRP still fails the LAN gateway over to `CAIRO-CORE-02`, but the HQ subnets will stop being advertised to the WAN until `CAIRO-CORE-01` recovers (or OSPF is manually added to `CORE-02`). This is a known, deliberate limitation, not a bug.

### 🔀 CAIRO-ACCESS-SW01 (Access Switch)

```text
hostname CAIRO-ACCESS-SW01
spanning-tree mode rapid-pvst
spanning-tree portfast default
spanning-tree portfast bpduguard default
ip dhcp snooping
ip dhcp snooping vlan 10,20
no ip dhcp snooping information option
vlan 10
 name DATA_HQ
vlan 20
 name VOICE_HQ
vlan 30
 name SERVERS_HQ
exit
interface GigabitEthernet0/1
 switchport trunk encapsulation dot1q
 switchport mode trunk
 ip dhcp snooping trust
exit
interface GigabitEthernet0/2
 switchport trunk encapsulation dot1q
 switchport mode trunk
 ip dhcp snooping trust
exit
interface FastEthernet0/1
 switchport mode access
 switchport access vlan 10
 switchport port-security
 switchport port-security maximum 2
 switchport port-security violation restrict
 switchport port-security mac-address sticky
exit
interface FastEthernet0/2
 switchport mode access
 switchport access vlan 10
 switchport port-security
 switchport port-security maximum 2
 switchport port-security violation restrict
 switchport port-security mac-address sticky
exit
interface FastEthernet0/3
 switchport mode access
 switchport access vlan 30
exit
interface range FastEthernet0/4-20
 switchport mode access
 switchport access vlan 10
 switchport port-security
 switchport port-security maximum 2
 switchport port-security violation restrict
 switchport port-security mac-address sticky
end
```

> `ip dhcp snooping vlan 10,20` only (VLAN 30 excluded since the server itself lives there and DHCP snooping shouldn't distrust the DHCP server's own segment). Alexandria's `110,120` were removed from this line since Alexandria has its own separate switch (`ALEX-SW01`) — DHCP snooping there was never configured because Alexandria clients only relay to the central server, they don't host one locally. The voice-VLAN mapping (`switchport voice vlan 20`) was also dropped since there are no IP phones in this topology.

### 🌐 CAIRO-EDGE-GW01 (Edge & Internet Router)

```text
hostname CAIRO-EDGE-GW01
interface GigabitEthernet0/0
 description Uplink to Cairo Core CAIRO-CORE-01
 ip address 192.168.100.2 255.255.255.252
 ip nat inside
 no shutdown
exit
interface GigabitEthernet0/0/0
 description Primary WAN Link to ALEX-GW-01
 ip address 10.100.1.1 255.255.255.252
 ip nat inside
 no shutdown
exit
interface GigabitEthernet0/1/0
 description Backup WAN Link to ALEX-GW-02
 ip address 10.100.2.1 255.255.255.252
 ip nat inside
 no shutdown
exit
interface GigabitEthernet0/2/0
 description ISP Connection
 ip address 203.0.113.1 255.255.255.252
 ip nat outside
 no shutdown
exit
router ospf 1
 router-id 1.1.1.1
 network 192.168.100.0 0.0.0.3 area 0
 network 10.100.1.0 0.0.0.3 area 0
 default-information originate
exit
ip route 172.16.0.0 255.255.0.0 10.100.2.2 210
ip route 0.0.0.0 0.0.0.0 203.0.113.2
access-list 100 permit ip 192.168.0.0 0.0.255.255 any
access-list 100 permit ip 172.16.0.0 0.0.255.255 any
ip nat inside source list 100 interface GigabitEthernet0/2/0 overload
end
```

> `GigabitEthernet0/0-0/2` (the router's 3 built-in copper ports) were **not** used for the WAN/ISP links — Packet Tracer requires router↔router links to be fiber. Only `Gi0/0` (to the Core switch) is copper; `Gi0/0/0`, `Gi0/1/0`, `Gi0/2/0` are fiber expansion ports. The ISP subnet also changed from `195.246.10.0/30` to `203.0.113.0/30` (a valid documentation/test range) and the default route now points to `203.0.113.2` (ISP-RTR), not `195.246.10.1`.

### 🌍 ISP-RTR (Simulated Internet)

```text
hostname ISP-RTR
interface GigabitEthernet0/0/0
 description Link to CAIRO-EDGE-GW01
 ip address 203.0.113.2 255.255.255.252
 no shutdown
exit
interface Loopback0
 description Simulated Internet Host
 ip address 8.8.8.8 255.255.255.255
end
```

> This device is new versus the original draft — the original just said "1x Router عام (ISP)" with no config. It's now a real 2911 with a `Loopback0 8.8.8.8/32`, which is the actual ping/traceroute target used in Test 4 below.

### 🚦 ALEX-GW-01 (Active Branch Router)

```text
hostname ALEX-GW-01
interface GigabitEthernet0/0
 no shutdown
exit
interface GigabitEthernet0/0.110
 description Alex Data Subnet
 encapsulation dot1Q 110
 ip address 172.16.10.2 255.255.255.0
 ip helper-address 192.168.30.10
 standby version 2
 standby 110 ip 172.16.10.1
 standby 110 priority 110
 standby 110 preempt
exit
interface GigabitEthernet0/0.120
 description Alex Voice Subnet
 encapsulation dot1Q 120
 ip address 172.16.20.2 255.255.255.0
 ip helper-address 192.168.30.10
 standby version 2
 standby 120 ip 172.16.20.1
 standby 120 priority 110
 standby 120 preempt
exit
interface GigabitEthernet0/0/0
 description Primary WAN Link to CAIRO-EDGE-GW01
 ip address 10.100.1.2 255.255.255.252
 no shutdown
exit
router ospf 1
 router-id 2.2.2.2
 network 172.16.10.0 0.0.0.255 area 0
 network 172.16.20.0 0.0.0.255 area 0
 network 10.100.1.0 0.0.0.3 area 0
end
```

> The WAN uplink moved from `Gi0/1` (copper) to `Gi0/0/0` (fiber) — same reason as above. The two OSPF `network 172.16.0.0 0.0.255.255` wildcard statements were split into precise `/24` statements for each VLAN, which is functionally identical but avoids accidentally matching future subnets.

### 🚦 ALEX-GW-02 (Standby Branch Router)

```text
hostname ALEX-GW-02
interface GigabitEthernet0/0
 no shutdown
exit
interface GigabitEthernet0/0.110
 encapsulation dot1Q 110
 ip address 172.16.10.3 255.255.255.0
 ip helper-address 192.168.30.10
 standby version 2
 standby 110 ip 172.16.10.1
 standby 110 priority 100
 standby 110 preempt
exit
interface GigabitEthernet0/0.120
 encapsulation dot1Q 120
 ip address 172.16.20.3 255.255.255.0
 ip helper-address 192.168.30.10
 standby version 2
 standby 120 ip 172.16.20.1
 standby 120 priority 100
 standby 120 preempt
exit
interface GigabitEthernet0/0/0
 description Backup WAN Link to CAIRO-EDGE-GW01
 ip address 10.100.2.2 255.255.255.252
 no shutdown
exit
ip route 192.168.0.0 255.255.0.0 10.100.2.1 210
end
```

### 🔀 ALEX-SW01 (Alexandria Access Switch)

```text
hostname ALEX-SW01
spanning-tree mode rapid-pvst
spanning-tree portfast default
spanning-tree portfast bpduguard default
vlan 110
 name DATA_ALEX
vlan 120
 name VOICE_ALEX
exit
interface GigabitEthernet0/1
 switchport trunk encapsulation dot1q
 switchport mode trunk
exit
interface GigabitEthernet0/2
 switchport trunk encapsulation dot1q
 switchport mode trunk
exit
interface FastEthernet0/1
 switchport mode access
 switchport access vlan 110
exit
interface FastEthernet0/2
 switchport mode access
 switchport access vlan 110
end
```

### 🖥️ Endpoints

- `CAIRO-PC1`, `CAIRO-PC2`, `ALEX-PC1`, `ALEX-PC2`: IP Configuration → **DHCP**.
- `CAIRO-DHCP-DNS`: static IP `192.168.30.10 / 255.255.255.0`, gateway `192.168.30.1`, DNS `192.168.30.10`. DHCP pools: `Cairo-Data` (192.168.10.0/24, gateway 192.168.10.1, start 192.168.10.50) and `Alex-Data` (172.16.10.0/24, gateway 172.16.10.1, start 172.16.10.50). DNS record: `www.company.local → 192.168.30.10`.

---

## 🛡️ Layer 2 Security Configuration

| Feature | Command / Setting | Purpose |
| :--- | :--- | :--- |
| **Rapid PVST+** | `spanning-tree mode rapid-pvst` | Fast convergence for loop prevention |
| **PortFast** | `spanning-tree portfast default` | Immediate Forwarding on access ports |
| **BPDU Guard** | `spanning-tree portfast bpduguard default` | Err-disable port if an unauthorized switch is detected |
| **DHCP Snooping** | `ip dhcp snooping vlan 10,20` | Filter rogue DHCP server responses on Cairo |
| **Port Security** | `switchport port-security maximum 2` | Limit MAC addresses per access port |
| **Sticky MAC** | `switchport port-security mac-address sticky` | Learn and lock MAC addresses dynamically |
| **Violation Restrict** | `switchport port-security violation restrict` | Drop violating traffic, keep the port up |
| **Trunk Trust** | `ip dhcp snooping trust` (trunks) | Allow DHCP relay traffic on trusted uplinks |

```
End-Device → Access Port (PortFast + BPDU Guard + Port Security)
                ↓
         Rogue DHCP Blocked (DHCP Snooping)
         Rogue Switch Blocked (BPDU Guard → err-disabled)
         Unauthorized MAC Blocked (Port Security → restrict)
                ↓
         Trunk Port (DHCP Snooping Trust)
                ↓
         Core Switch → DHCP Relay → Central DHCP Server
```

---

## 🌐 NAT/PAT Configuration

```text
interface GigabitEthernet0/0
 ip nat inside
interface GigabitEthernet0/0/0
 ip nat inside
interface GigabitEthernet0/1/0
 ip nat inside
interface GigabitEthernet0/2/0
 ip nat outside

access-list 100 permit ip 192.168.0.0 0.0.255.255 any   ! Cairo HQ
access-list 100 permit ip 172.16.0.0 0.0.255.255 any     ! Alexandria Branch

ip nat inside source list 100 interface GigabitEthernet0/2/0 overload
ip route 0.0.0.0 0.0.0.0 203.0.113.2
```

> All three internal-facing interfaces (`Gi0/0` to Core, `Gi0/0/0` and `Gi0/1/0` to Alexandria) are marked `ip nat inside` so traffic from **either** branch gets translated, not just Cairo's LAN.

```
Cairo PC (192.168.10.50) → CAIRO-CORE-01 (HSRP VIP) → CAIRO-EDGE-GW01
                                                             ↓
Alex PC (172.16.10.50) → ALEX-GW-01 (HSRP VIP) → OSPF WAN → CAIRO-EDGE-GW01
                                                             ↓
                                                    NAT/PAT Translation
                                                             ↓
                                              ISP-RTR (203.0.113.2) → 8.8.8.8
```

---

## ✅ Verification & Testing Scenarios

### Test 1 — LACP EtherChannel
```
show etherchannel summary
```
Expected: `Po1(SU)` with `Gi0/1(P)` and `Gi0/2(P)` — both member ports bundled and forwarding.

### Test 2 — HSRP Gateway Redundancy
```
show standby brief
```
Expected: `CAIRO-CORE-01` Active (110), `CAIRO-CORE-02` Standby (100).

Failover test — from `CAIRO-PC1`:
```
ping 192.168.10.1 -t
```
On `CAIRO-CORE-01`:
```
configure terminal
interface Vlan10
 shutdown
```
Expected: 1–2 lost pings, then traffic resumes via `CAIRO-CORE-02`. Re-enable with `no shutdown` afterward.

### Test 3 — DHCP Relay Across WAN
On `ALEX-PC1`: IP Configuration → DHCP.
Expected: address from `172.16.10.50+`, gateway `172.16.10.1`, DNS `192.168.30.10` — proves the request reached the Cairo server through `ip helper-address` over the primary WAN link.

### Test 4 — OSPF, NAT/PAT and Internet Reachability
From `CAIRO-PC1`:
```
ping 8.8.8.8
```
Expected: success (validates NAT overload on `CAIRO-EDGE-GW01`).

From `ALEX-PC1`:
```
tracert 192.168.30.10
```
Expected path: `ALEX-GW-01 → 10.100.1.1 (CAIRO-EDGE-GW01) → 192.168.100.1 (CAIRO-CORE-01) → 192.168.30.10`.

### Test 5 — Floating Static Route (Primary WAN Failure)
On `CAIRO-EDGE-GW01`:
```
configure terminal
interface GigabitEthernet0/0/0
 shutdown
```
Wait ~30–40 seconds (OSPF dead-timer expiry), then:
```
show ip route
```
Expected: the OSPF route to `172.16.0.0/16` disappears, replaced by the static route (AD 210) via `10.100.2.2`.

From `ALEX-PC2`:
```
ping 192.168.30.10
```
Expected: succeeds over the backup path despite the primary being down. Restore with `no shutdown` on `Gi0/0/0` afterward.

### Test 6 — Layer 2 Security (BPDU Guard & Port Security)
Connect an extra switch to `CAIRO-ACCESS-SW01 Fa0/1`:
```
show interfaces status
```
Expected: the port shows `err-disabled`. Recover with `shutdown` then `no shutdown` on that interface.

Connect a second and then a third device to one access port:
```
show port-security interface FastEthernet0/1
```
Expected: the third device is denied (violation restrict) while the first two keep working (max 2).

---

## ⚡ How to Run Lab

1.  **Open the `.pkt` file** in Cisco Packet Tracer — all configs above are already pre-applied.
2.  **Wait ~30 seconds** after opening for EtherChannel, HSRP, and OSPF to converge before testing.
3.  **Run the tests** from the [Verification & Testing Scenarios](#-verification--testing-scenarios) section above, in order.
4.  If you rebuild from scratch instead of using the exported file, paste the CLI blocks above into each device in the order shown, then `write memory` / `copy running-config startup-config` on every network device.

---

## 🎓 Learning Outcomes

1.  Designed and implemented a multi-site enterprise network with redundancy at the gateway, WAN, and link levels.
2.  Configured HSRP v2 for gateway failover with explicit Active/Standby priorities.
3.  Implemented LACP EtherChannel for link aggregation between core switches.
4.  Deployed OSPF Area 0 with a Floating Static Route for automatic WAN failover.
5.  Centralized DHCP services using DHCP Relay across sites.
6.  Applied Layer 2 security hardening: BPDU Guard, Port Security (sticky MAC), DHCP Snooping.
7.  Configured NAT/PAT for Internet access across multiple internal subnets, verified against a simulated ISP.
8.  Practiced Packet Tracer-specific constraints: fiber-only router↔router links, 2-port GigE limits on 3560 switches, and asymmetric-routing avoidance on redundant L3 switches.

---

## 💡 Repository Info

- **Platform:** Cisco Packet Tracer (9.0 recommended)
- **Focus Areas:** HSRP v2 • LACP EtherChannel • OSPF • Floating Static Routes • DHCP Relay • Layer 2 Security • NAT/PAT • Rapid PVST+
- **Architecture:** 3-Tier (Cairo HQ) + Dual-Gateway (Alexandria Branch)
- **Sites:** Cairo HQ (192.168.0.0/16) + Alexandria Branch (172.16.0.0/16)
- **Device count:** 13 (2 core switches, 1 access switch, 1 edge router, 1 ISP router, 1 server, 2 PCs — Cairo; 2 gateway routers, 1 switch, 2 PCs — Alexandria)
