# 🏢 Multi-Site Enterprise Network — Cairo HQ & Alexandria Branch

![Cisco Packet Tracer](https://img.shields.io/badge/Tool-Cisco%20Packet%20Tracer-blue?logo=cisco)
![HSRP v2](https://img.shields.io/badge/Protocol-HSRP%20v2-green)
![OSPF](https://img.shields.io/badge/Protocol-OSPF%20Area%200-brightgreen)
![LACP](https://img.shields.io/badge/Protocol-LACP%20EtherChannel-orange)
![Network Design](https://img.shields.io/badge/Focus-Enterprise%20Network%20Design-red)
![Layer 3 Switching](https://img.shields.io/badge/Layer-L3%20Switching-purple)
![High Availability](https://img.shields.io/badge/Feature-High%20Availability-success)
![Status](https://img.shields.io/badge/Status-Completed-success)

---

## 📑 Table of Contents

1.  [📘 Project Overview](#-project-overview)
2.  [🎯 Project Objectives](#-project-objectives)
3.  [🌐 Network Topology](#-network-topology)
4.  [🔗 Device Interface Table](#-device-interface-table)
5.  [📝 IP Addressing Table](#-ip-addressing-table)
6.  [🔧 Implementation Steps](#-implementation-steps)
7.  [💻 Device Configuration](#-device-configuration)
    - [🏗️ CAIRO-CORE-01 (Active Core Switch)](#-cairo-core-01-active-core-switch)
    - [🏗️ CAIRO-CORE-02 (Standby Core Switch)](#-cairo-core-02-standby-core-switch)
    - [🚦 ALEX-GW-01 (Active Branch Router)](#-alex-gw-01-active-branch-router)
    - [🚦 ALEX-GW-02 (Standby Branch Router)](#-alex-gw-02-standby-branch-router)
    - [🌐 CAIRO-EDGE-GW01 (Edge & Internet Router)](#-cairo-edge-gw01-edge--internet-router)
    - [🔀 CAIRO-ACCESS-SW01 (Access Switch)](#-cairo-access-sw01-access-switch)
8.  [🛡️ Layer 2 Security Configuration](#-layer-2-security-configuration)
9.  [🌐 NAT/PAT Configuration](#-natpat-configuration)
10. [✅ Verification & Testing Scenarios](#-verification--testing-scenarios)
11. [⚡ How to Run Lab](#-how-to-run-lab)
12. [📂 Folder Structure](#-folder-structure)
13. [🧱 Lab Limitations](#-lab-limitations)
14. [🎓 Learning Outcomes](#-learning-outcomes)
15. [💡 Repository Info](#-repository-info)

---

## 📘 Project Overview

This lab demonstrates a **multi-site enterprise network** connecting two branches (**Cairo HQ** and **Alexandria Branch**) with **full redundancy**, **centralized services**, and **end-to-end security**.

The design integrates:
- **HSRP v2** for gateway redundancy at both sites
- **LACP EtherChannel** for link aggregation between core switches
- **OSPF Area 0** for primary WAN routing with **Floating Static Routes** for backup path
- **Centralized DHCP** via DHCP Relay across WAN
- **Layer 2 Security Hardening** (BPDU Guard, Port Security, DHCP Snooping)
- **NAT/PAT** for controlled Internet access
- **Rapid PVST+** for loop-free Layer 2 topology

The architecture guarantees **Zero Single Point of Failure** across all critical paths.

---

## 🎯 Project Objectives

The key goals of this project:

1.  Design a **3-Tier Architecture** (Core / Distribution / Access) for Cairo HQ and a **Collapsed Core** model for Alexandria Branch.
2.  Deploy **HSRP v2** between CAIRO-CORE-01/02 and ALEX-GW-01/02 for gateway redundancy.
3.  Implement **LACP EtherChannel** (Port-channel 1) between core switches for increased bandwidth and link redundancy.
4.  Configure **OSPF Area 0** on the primary WAN link with a **Floating Static Route** (AD 210) as backup.
5.  Centralize **DHCP services** in Cairo and distribute addresses to both sites via **DHCP Relay (ip helper-address)**.
6.  Apply **Layer 2 security hardening**: BPDU Guard, Port Security (sticky MAC), DHCP Snooping, and ARP protection.
7.  Enable **NAT/PAT** on the edge router for Internet access for both branches.
8.  Validate failover behavior across **gateway**, **WAN link**, and **port security** scenarios.

---

## 🌐 Network Topology

📸 **Network Topology Diagram**  
`/topology/topology_overview.png`

<img src="topology/topology_overview.png" alt="NETWORK TOPOLOGY" style="border:1px solid #ddd; padding:5px; max-width:100%; height:auto;">

A simplified overview of the design:

- **Cairo HQ (3-Tier)**
  - **Core Layer:** CAIRO-CORE-01 (Active) & CAIRO-CORE-02 (Standby) — L3 Multilayer Switches connected via LACP EtherChannel
  - **Access Layer:** CAIRO-ACCESS-SW01 — with Port Security and BPDU Guard
  - **Edge:** CAIRO-EDGE-GW01 — OSPF + Floating Static + NAT/PAT + ISP link
- **Alexandria Branch (Collapsed Core)**
  - **Branch Gateways:** ALEX-GW-01 (Active) & ALEX-GW-02 (Standby) — Router-on-a-Stick with HSRP v2
- **WAN Links:**
  - **Primary:** 10.100.1.0/30 — OSPF Area 0 (CAIRO-EDGE-GW01 ↔ ALEX-GW-01)
  - **Backup:** 10.100.2.0/30 — Floating Static Route AD 210 (CAIRO-EDGE-GW02 ↔ ALEX-GW-02)
- **Internet:** 195.246.10.0/30 — ISP connection via CAIRO-EDGE-GW01
- **Centralized Services:** DHCP/DNS Server at 192.168.30.10 (VLAN 30, Cairo)

---

## 🔗 Device Interface Table

| Device             | Interface         | Connection                  | Description                              |
| :----------------- | :---------------- | :-------------------------- | :--------------------------------------- |
| **CAIRO-CORE-01**  | G1/0/1 - G1/0/2   | CAIRO-CORE-02 G1/0/1-2     | LACP EtherChannel (Port-channel 1)      |
| **CAIRO-CORE-01**  | G1/0/24           | Tracked by HSRP             | Interface tracking for priority decrement|
| **CAIRO-CORE-02**  | G1/0/1 - G1/0/2   | CAIRO-CORE-01 G1/0/1-2     | LACP EtherChannel (Port-channel 1)      |
| **CAIRO-EDGE-GW01**| G0/0              | ALEX-GW-01 G0/1            | Primary WAN (OSPF Area 0) — 10.100.1.0/30|
| **CAIRO-EDGE-GW01**| G0/1              | ALEX-GW-02 G0/1            | Backup WAN (Floating Static) — 10.100.2.0/30|
| **CAIRO-EDGE-GW01**| G0/2              | ISP                         | Internet (NAT Outside) — 195.246.10.0/30|
| **ALEX-GW-01**     | G0/0              | Alex Access Switch          | Router-on-a-Stick (Sub-interfaces 110, 120)|
| **ALEX-GW-01**     | G0/1              | CAIRO-EDGE-GW01 G0/0       | Primary WAN Link — 10.100.1.0/30        |
| **ALEX-GW-02**     | G0/0              | Alex Access Switch          | Router-on-a-Stick (Sub-interfaces 110, 120)|
| **ALEX-GW-02**     | G0/1              | CAIRO-EDGE-GW01 G0/1       | Backup WAN Link — 10.100.2.0/30         |
| **CAIRO-ACCESS-SW01**| G0/1 - G0/2    | Core Switches (Trunks)      | 802.1Q Trunk + DHCP Snooping Trust       |
| **CAIRO-ACCESS-SW01**| F0/1 - F0/20   | End-user Devices            | Access Ports (VLAN 10 + Voice VLAN 20)   |

---

## 📝 IP Addressing Table

### Cairo HQ — 192.168.0.0/16

| Network / Component    | Subnet / Address    | Gateway / Purpose                  |
| :--------------------- | :------------------ | :--------------------------------- |
| **Data Network**       | 192.168.10.0 /24    | HSRP Virtual IP: 192.168.10.1     |
| **Voice Network**      | 192.168.20.0 /24    | HSRP Virtual IP: 192.168.20.1     |
| **Servers Network**    | 192.168.30.0 /24    | HSRP Virtual IP: 192.168.30.1     |
| **Management Network** | 192.168.99.0 /24    | HSRP Virtual IP: 192.168.99.1     |
| **DHCP/DNS Server**    | 192.168.30.10       | Centralized DHCP + DNS Service     |
| **Primary WAN Link**   | 10.100.1.0 /30      | OSPF Area 0                        |
| **Backup WAN Link**    | 10.100.2.0 /30      | Floating Static (AD 210)           |
| **ISP Link**           | 195.246.10.0 /30    | NAT Outside / Internet             |

### Alexandria Branch — 172.16.0.0/16

| Network / Component    | Subnet / Address    | Gateway / Purpose                  |
| :--------------------- | :------------------ | :--------------------------------- |
| **Alex Data Network**  | 172.16.10.0 /24     | HSRP Virtual IP: 172.16.10.1      |
| **Alex Voice Network** | 172.16.20.0 /24     | HSRP Virtual IP: 172.16.20.1      |
| **Alex Mgmt Network**  | 172.16.99.0 /24     | HSRP Virtual IP: 172.16.99.1      |

### VLAN Summary

| VLAN ID | Name        | Network        | Mask           | Virtual Gateway | Site         | Purpose                       |
| :-----: | :---------- | :------------- | :------------- | :-------------- | :----------- | :---------------------------- |
| **10**  | DATA_HQ     | 192.168.10.0   | 255.255.255.0  | 192.168.10.1    | Cairo        | End-user Data Traffic         |
| **20**  | VOICE_HQ    | 192.168.20.0   | 255.255.255.0  | 192.168.20.1    | Cairo        | VoIP Traffic                  |
| **30**  | SERVERS_HQ  | 192.168.30.0   | 255.255.255.0  | 192.168.30.1    | Cairo        | DHCP/DNS Server + Infrastructure|
| **99**  | MGMT_HQ     | 192.168.99.0   | 255.255.255.0  | 192.168.99.1    | Cairo        | Network Management Access     |
| **110** | ALEX_DATA   | 172.16.10.0    | 255.255.255.0  | 172.16.10.1     | Alexandria   | Branch Data Traffic           |
| **120** | ALEX_VOICE  | 172.16.20.0    | 255.255.255.0  | 172.16.20.1     | Alexandria   | Branch VoIP Traffic           |
| **199** | ALEX_MGMT   | 172.16.99.0    | 255.255.255.0  | 172.16.99.1     | Alexandria   | Branch Management Access      |

### HSRP Configuration Summary

| VLAN / Group | Virtual IP      | Active Device    | Active Priority | Standby Device   | Standby Priority | Tracking                  |
| :-----------: | :-------------- | :--------------- | :--------------: | :--------------- | :---------------: | :------------------------ |
| **10**       | 192.168.10.1    | CAIRO-CORE-01    | 110             | CAIRO-CORE-02    | 100              | G1/0/24 (decrement 20)    |
| **20**       | 192.168.20.1    | CAIRO-CORE-01    | 110             | CAIRO-CORE-02    | 100              | —                         |
| **30**       | 192.168.30.1    | CAIRO-CORE-01    | 110             | CAIRO-CORE-02    | 100              | —                         |
| **110**      | 172.16.10.1     | ALEX-GW-01       | 110             | ALEX-GW-02       | 100              | G0/1 (decrement 20)       |
| **120**      | 172.16.20.1     | ALEX-GW-01       | 110             | ALEX-GW-02       | 100              | —                         |

---

## 🔧 Implementation Steps

1.  **Core Layer Redundancy (Cairo HQ)**  
    Created LACP EtherChannel (Port-channel 1) between CAIRO-CORE-01 and CAIRO-CORE-02 on G1/0/1-2.  
    Configured HSRP v2 on all SVIs with CAIRO-CORE-01 as Active (priority 110) and CAIRO-CORE-02 as Standby (priority 100).  
    Set CAIRO-CORE-01 as Primary Root Bridge and CAIRO-CORE-02 as Secondary Root Bridge for all VLANs.

2.  **Branch Gateway Redundancy (Alexandria)**  
    Configured ALEX-GW-01 and ALEX-GW-02 with Router-on-a-Stick sub-interfaces for VLAN 110 (Data) and VLAN 120 (Voice).  
    Deployed HSRP v2 with ALEX-GW-01 as Active (priority 110, tracking G0/1).

3.  **WAN Connectivity (OSPF + Floating Static)**  
    Configured OSPF Area 0 on CAIRO-EDGE-GW01 and ALEX-GW-01 over the primary WAN link (10.100.1.0/30).  
    Added Floating Static Route (AD 210) on CAIRO-EDGE-GW01 via the backup link (10.100.2.0/30) for automatic failover.

4.  **Centralized DHCP Service**  
    Placed DHCP/DNS server at 192.168.30.10 in Cairo (VLAN 30).  
    Configured `ip helper-address 192.168.30.10` on all SVIs and sub-interfaces to relay DHCP requests across sites.

5.  **Layer 2 Security Hardening**  
    Enabled Rapid PVST+, PortFast, and BPDU Guard on all access switches.  
    Configured DHCP Snooping, Port Security (sticky MAC, max 2, violation restrict) on user ports.

6.  **Internet Access (NAT/PAT)**  
    Defined NAT inside/outside interfaces on CAIRO-EDGE-GW01.  
    Applied PAT overload using ACL 100 for both Cairo (192.168.0.0/16) and Alexandria (172.16.0.0/16) subnets.  
    Set default route toward ISP (195.246.10.1).

---

## 💻 Device Configuration

📁 All configurations are available in the `configs/` folder.

---

### 🏗️ CAIRO-CORE-01 (Active Core Switch)

```text
! --- Hostname & Routing ---
hostname CAIRO-CORE-01
ip routing

! --- LACP EtherChannel (Port-channel 1) ---
interface range GigabitEthernet1/0/1 - 2
 switchport trunk encapsulation dot1q
 switchport mode trunk
 channel-group 1 mode active

interface Port-channel1
 switchport trunk encapsulation dot1q
 switchport mode trunk

! --- VLAN Creation ---
vlan 10
 name DATA_HQ
vlan 20
 name VOICE_HQ
vlan 30
 name SERVERS_HQ
vlan 99
 name MGMT_HQ

! --- SVI: Data VLAN (HSRP Active) ---
interface Vlan10
 description Data VLAN Gateway
 ip address 192.168.10.2 255.255.255.0
 ip helper-address 192.168.30.10
 standby version 2
 standby 10 ip 192.168.10.1
 standby 10 priority 110
 standby 10 preempt
 standby 10 track GigabitEthernet1/0/24 20
```

[View Full Configuration File →](configs/switch-config/cairo-core-01.cfg)

---

### 🏗️ CAIRO-CORE-02 (Standby Core Switch)

```text
! --- Hostname & Routing ---
hostname CAIRO-CORE-02
ip routing

! --- LACP EtherChannel (Port-channel 1) ---
interface range GigabitEthernet1/0/1 - 2
 switchport trunk encapsulation dot1q
 switchport mode trunk
 channel-group 1 mode active

interface Port-channel1
 switchport trunk encapsulation dot1q
 switchport mode trunk

! --- SVI: Data VLAN (HSRP Standby) ---
interface Vlan10
 ip address 192.168.10.3 255.255.255.0
 ip helper-address 192.168.30.10
 standby version 2
 standby 10 ip 192.168.10.1
 standby 10 priority 100
 standby 10 preempt

! --- Spanning Tree: Secondary Root ---
spanning-tree mode rapid-pvst
spanning-tree vlan 10,20,30,99 root secondary
```

[View Full Configuration File →](configs/switch-config/cairo-core-02.cfg)

---

### 🚦 ALEX-GW-01 (Active Branch Router)

```text
hostname ALEX-GW-01

! --- Router-on-a-Stick: Data Sub-interface (VLAN 110) ---
interface GigabitEthernet0/0.110
 description Alex Data Subnet
 encapsulation dot1Q 110
 ip address 172.16.10.2 255.255.255.0
 ip helper-address 192.168.30.10
 standby version 2
 standby 110 ip 172.16.10.1
 standby 110 priority 110
 standby 110 preempt
 standby 110 track GigabitEthernet0/1 20

! --- Router-on-a-Stick: Voice Sub-interface (VLAN 120) ---
interface GigabitEthernet0/0.120
 description Alex Voice Subnet
 encapsulation dot1Q 120
 ip address 172.16.20.2 255.255.255.0
 ip helper-address 192.168.30.10
 standby version 2
 standby 120 ip 172.16.20.1
 standby 120 priority 110
 standby 120 preempt

! --- Primary WAN Link (OSPF) ---
interface GigabitEthernet0/1
 description Primary WAN Link to Cairo
 ip address 10.100.1.2 255.255.255.252
 no shutdown

! --- OSPF Area 0 ---
router ospf 1
 router-id 2.2.2.2
 network 172.16.0.0 0.0.255.255 area 0
 network 10.100.1.0 0.0.0.3 area 0
```

[View Full Configuration File →](configs/router-config/alex-gw-01.cfg)

---

### 🚦 ALEX-GW-02 (Standby Branch Router)

```text
hostname ALEX-GW-02

! --- Router-on-a-Stick: Data Sub-interface (VLAN 110) ---
interface GigabitEthernet0/0.110
 encapsulation dot1Q 110
 ip address 172.16.10.3 255.255.255.0
 ip helper-address 192.168.30.10
 standby version 2
 standby 110 ip 172.16.10.1
 standby 110 priority 100
 standby 110 preempt

! --- Router-on-a-Stick: Voice Sub-interface (VLAN 120) ---
interface GigabitEthernet0/0.120
 encapsulation dot1Q 120
 ip address 172.16.20.3 255.255.255.0
 ip helper-address 192.168.30.10
 standby version 2
 standby 120 ip 172.16.20.1
 standby 120 priority 100
 standby 120 preempt
```

[View Full Configuration File →](configs/router-config/alex-gw-02.cfg)

---

### 🌐 CAIRO-EDGE-GW01 (Edge & Internet Router)

```text
hostname CAIRO-EDGE-GW01

! --- Primary WAN Link to Alexandria (OSPF Area 0) ---
interface GigabitEthernet0/0
 description Primary WAN Link to ALEX
 ip address 10.100.1.1 255.255.255.252
 ip nat inside
 no shutdown

! --- Backup WAN Link to Alexandria (Floating Static) ---
interface GigabitEthernet0/1
 description Backup WAN Link to ALEX
 ip address 10.100.2.1 255.255.255.252
 no shutdown

! --- ISP Internet Connection (NAT Outside) ---
interface GigabitEthernet0/2
 description ISP Connection
 ip address 195.246.10.2 255.255.255.252
 ip nat outside
 no shutdown

! --- OSPF Area 0 Configuration ---
router ospf 1
 router-id 1.1.1.1
 network 192.168.0.0 0.0.255.255 area 0
 network 10.100.1.0 0.0.0.3 area 0
 default-information originate

! --- Floating Static Route (Backup WAN) ---
! Administrative Distance 210 ensures OSPF (AD 110) is preferred
ip route 172.16.0.0 255.255.0.0 10.100.2.2 210

! --- Default Route to ISP ---
ip route 0.0.0.0 0.0.0.0 195.246.10.1
```

[View Full Configuration File →](configs/router-config/cairo-edge-gw01.cfg)

---

### 🔀 CAIRO-ACCESS-SW01 (Access Switch)

```text
hostname CAIRO-ACCESS-SW01

! --- Spanning Tree & BPDU Guard ---
spanning-tree mode rapid-pvst
spanning-tree portfast default
spanning-tree portfast bpduguard default

! --- DHCP Snooping ---
ip dhcp snooping
ip dhcp snooping vlan 10,20,110,120
no ip dhcp snooping information option

! --- Access Ports: User Devices (VLAN 10 + Voice VLAN 20) ---
interface range FastEthernet0/1 - 20
 switchport mode access
 switchport access vlan 10
 switchport voice vlan 20
 switchport port-security
 switchport port-security maximum 2
 switchport port-security violation restrict
 switchport port-security mac-address sticky
 switchport dhcp snooping trust

! --- Trunk Ports to Core ---
interface range GigabitEthernet0/1 - 2
 switchport mode trunk
 ip dhcp snooping trust
```

[View Full Configuration File →](configs/switch-config/cairo-access-sw01.cfg)

---

## 🛡️ Layer 2 Security Configuration

This project implements **comprehensive Layer 2 security** across all access switches in both branches.

### Security Features Applied

| Feature                  | Command / Setting                     | Purpose                                         |
| :---------------------- | :------------------------------------ | :---------------------------------------------- |
| **Rapid PVST+**         | `spanning-tree mode rapid-pvst`       | Fast convergence (sub-second) for loop prevention|
| **PortFast**            | `spanning-tree portfast default`      | Immediate transition to Forwarding on access ports|
| **BPDU Guard**          | `spanning-tree portfast bpduguard default` | Err-disable port if unauthorized switch detected |
| **DHCP Snooping**       | `ip dhcp snooping vlan 10,20,110,120`| Filter rogue DHCP server responses              |
| **Port Security**       | `switchport port-security maximum 2`  | Limit MAC addresses per port (PC + IP Phone)    |
| **Sticky MAC**          | `switchport port-security mac-address sticky` | Dynamically learn and lock MAC addresses   |
| **Violation Restrict**  | `switchport port-security violation restrict` | Drop violating traffic + SNMP trap (no shutdown) |
| **Trunk Trust**         | `ip dhcp snooping trust` (trunks)     | Allow DHCP relay traffic on trusted uplinks      |

### Security Flow Summary

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

Internet access is provided through **CAIRO-EDGE-GW01** using **PAT (Port Address Translation)**.

### NAT Design

```text
! --- Inside Interfaces (Internal Traffic) ---
interface GigabitEthernet0/0
 ip nat inside

! --- Outside Interface (ISP / Internet) ---
interface GigabitEthernet0/2
 ip nat outside

! --- ACL: Permit both branches to access Internet ---
access-list 100 permit ip 192.168.0.0 0.0.255.255 any   ! Cairo HQ
access-list 100 permit ip 172.16.0.0 0.0.255.255 any     ! Alexandria Branch

! --- PAT Overload on ISP Interface ---
ip nat inside source list 100 interface GigabitEthernet0/2 overload

! --- Default Route to ISP ---
ip route 0.0.0.0 0.0.0.0 195.246.10.1
```

### Traffic Flow (Internet Access)

```
Cairo PC (192.168.10.50) → CAIRO-CORE-01 (HSRP VIP) → CAIRO-EDGE-GW01
                                                             ↓
Alex PC (172.16.10.50) → ALEX-GW-01 (HSRP VIP) → OSPF WAN → CAIRO-EDGE-GW01
                                                             ↓
                                                    NAT/PAT Translation
                                                             ↓
                                                    ISP (195.246.10.1) → Internet
```

---

## ✅ Verification & Testing Scenarios

### Test 1: HSRP Gateway Redundancy

**Objective:** Verify automatic gateway failover when the active core switch fails.

```text
! Step 1: Verify HSRP state before failure
show standby brief

! Step 2: Continuous ping from a Cairo PC to the virtual gateway
ping 192.168.10.1 -t

! Step 3: Simulate failure on CAIRO-CORE-01
CAIRO-CORE-01# shutdown interface Vlan10

! Step 4: Verify HSRP state after failure
show standby brief

! Expected: CAIRO-CORE-02 transitions from Standby → Active (1-3 seconds, 1-2 packets lost)
```

### Test 2: WAN Failover (OSPF → Floating Static)

**Objective:** Verify traffic reroutes to backup WAN when the primary link fails.

```text
! Step 1: Traceroute from Alexandria PC to Cairo Server
traceroute 192.168.30.10

! Step 2: Simulate primary WAN failure
CAIRO-EDGE-GW01# shutdown interface GigabitEthernet0/0

! Step 3: Verify routing table — Floating Static should now be active
show ip route

! Step 4: Verify connectivity still works via backup path
traceroute 192.168.30.10

! Expected: OSPF route withdrawn → Floating Static (AD 210) installed via 10.100.2.2
```

### Test 3: BPDU Guard & Port Security

**Objective:** Verify Layer 2 security blocks unauthorized devices.

```text
! Step 1: Check port security status
show port-security interface fastEthernet0/1

! Step 2: Connect an unauthorized switch to an access port
! Expected: BPDU Guard immediately err-disables the port

! Step 3: Verify port status
show interfaces status | include err-disabled
show spanning-tree inconsistentports

! Step 4: Re-enable the port after investigation
CAIRO-ACCESS-SW01# shutdown
CAIRO-ACCESS-SW01# no shutdown
```

### Test 4: LACP EtherChannel Verification

**Objective:** Verify link aggregation between core switches.

```text
! Verify EtherChannel summary
show etherchannel summary

! Verify Port-channel status
show interfaces port-channel 1

! Expected: Port-channel1 is up, (SU) - Layer 2, in use, bundled (2 links)
```

### Test 5: DHCP Relay Across WAN

**Objective:** Verify Alexandria clients receive IP addresses from Cairo DHCP server.

```text
! On an Alexandria PC
ipconfig /all

! Expected: IP from 172.16.10.0/24 pool, Gateway 172.16.10.1 (HSRP VIP), DNS 192.168.30.10

! Verify DHCP relay on ALEX-GW-01
show ip helper-address
```

---

## ⚡ How to Run Lab

1.  **Open Topology:** Launch the `.pkt` file in Cisco Packet Tracer.
2.  **Load Configs:** Paste `.cfg` configurations into device CLIs from the `configs/` folder.
3.  **Run Tests:** Execute the verification commands from the [Verification & Testing Scenarios](#-verification--testing-scenarios) section above.
4.  **Review Screenshots:** Compare your results with those in `/screenshots/`.

---

## 📂 Folder Structure


enterprise-multi-site-ha/
├── configs/
│   ├── router-config/
│   │   ├── cairo-edge-gw01.cfg
│   │   ├── alex-gw-01.cfg
│   │   └── alex-gw-02.cfg
│   ├── switch-config/
│   │   ├── cairo-core-01.cfg
│   │   ├── cairo-core-02.cfg
│   │   └── cairo-access-sw01.cfg
│   └── security/
│       └── access-security.cfg
│
├── topology/
│   ├── topology_overview.png
│   └── topology_overview.drawio
│
├── lab-file/
│   └── enterprise-cairo-alex.pkt
│
├── screenshots/
│   ├── V1.1-Core_EtherChannel.png
│   ├── V1.2-HSRP_Active_Standby.png
│   ├── V1.3-HSRP_Failover.png
│   ├── V2.1-OSPF_Neighbors.png
│   ├── V2.2-WAN_Failover_Routing.png
│   ├── V3.1-DHCP_Relay_Alex.png
│   ├── V3.2-NAT_Translation.png
│   ├── V4.1-BPDU_Guard_Test.png
│   └── V4.2-Port_Security.png
│
├── README.md
└── verification.md

---

## 🧱 Lab Limitations

- **OSPF LSA Propagation in Packet Tracer:**
  Cisco Packet Tracer has limited OSPF support. Full LSA database inspection and advanced OSPF features (stub areas, virtual links) may not behave identically to real IOS devices.

- **Floating Static Route Convergence:**
  The OSPF dead interval must expire before the floating static route is installed. In lab environments, this may take 40 seconds (default hello 10s × 4). Tuning OSPF timers can reduce this for demonstration purposes.

- **DHCP Relay Across WAN:**
  Packet Tracer may require `ip helper-address` to be configured on every hop in the path. Ensure all intermediate routing interfaces properly forward UDP ports 67/68.

- **HSRP v2 Support:**
  HSRP v2 uses multicast address 224.0.0.102 (vs 224.0.0.2 for v1). Verify Packet Tracer version supports HSRP v2 before deploying.

---

## 🎓 Learning Outcomes

1.  Designed and implemented a **multi-site enterprise network** with full redundancy across gateway, WAN, and link levels.
2.  Configured **HSRP v2** for gateway failover with interface tracking and priority-based active/standby roles.
3.  Implemented **LACP EtherChannel** for link aggregation and increased bandwidth between core switches.
4.  Deployed **OSPF Area 0** with **Floating Static Routes** for automatic WAN failover.
5.  Centralized **DHCP services** using DHCP Relay across sites.
6.  Applied **Layer 2 security hardening**: BPDU Guard, Port Security (sticky MAC), DHCP Snooping.
7.  Configured **NAT/PAT** for Internet access across multiple internal subnets.
8.  Strengthened skills in **enterprise network design**, **troubleshooting**, and **high availability architecture**.

---

## 💡 Repository Info

- **Repository Name:** enterprise-multi-site-cairo-alexandria-ha
- **Platform:** Cisco Packet Tracer (Recommended: Latest Version)
- **Focus Areas:** HSRP v2 • LACP EtherChannel • OSPF • Floating Static Routes • DHCP Relay • Layer 2 Security • NAT/PAT • Rapid PVST+
- **Architecture:** 3-Tier (Cairo HQ) + Collapsed Core (Alexandria Branch)
- **Sites:** Cairo HQ (192.168.0.0/16) + Alexandria Branch (172.16.0.0/16)

---
