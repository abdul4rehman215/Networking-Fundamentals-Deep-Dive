# 🏷️ VLANs — Full Deep Dive

> **Source:** Professor Messer | **Topic #:** 11 | **Module:** 2 | **Status:** ✅ Complete

---

## 1. Definition

A **VLAN (Virtual Local Area Network)** is a logical grouping of network devices on one or more physical switches that communicate as if they were on the same isolated network — regardless of their physical location. Devices in different VLANs cannot communicate directly without going through a Layer 3 device (router or Layer 3 switch).

**Core goals:**
- **Security** — isolate sensitive departments from each other
- **Performance** — reduce broadcast domains, minimize unnecessary traffic
- **Flexibility** — group devices logically, not physically
- **Scalability** — easily restructure network without rewiring

> Think of VLANs as invisible walls inside a building. Everyone shares the same hallways (switch) and elevator (uplink), but each department (VLAN) is locked behind its own door — you can only cross between departments through reception (the router), where access is checked.

---

## 2. How VLANs Work — Step by Step

```
Step 1 — Network admin assigns VLAN IDs to switch ports
          Port 1-8   → VLAN 10 (Finance)
          Port 9-16  → VLAN 20 (HR)
          Port 17-24 → VLAN 30 (IT)

Step 2 — Device connects to port → automatically placed in that VLAN
          Device doesn't know it's on a VLAN — transparent to the host

Step 3 — Device sends a frame:
          ├── Destination in SAME VLAN → switch forwards directly
          └── Destination in DIFFERENT VLAN → switch drops it
                  (must go through router/Layer 3 switch)

Step 4 — Frame leaving the switch on a trunk port:
          → Switch adds 802.1Q tag (4-byte VLAN identifier)
          → Tag tells the next switch which VLAN this frame belongs to

Step 5 — Frame arrives at destination switch:
          → Switch reads 802.1Q tag
          → Strips tag before delivering to end device
          → End device never sees the VLAN tag
```

---

## 3. Key VLAN Concepts

### 3.1 VLAN ID
Every VLAN is identified by a number:
```
Range 1–4094 (12-bit field in 802.1Q header)
  VLAN 1      →  Default VLAN — all ports assigned here by default
  VLAN 2–1001 →  Normal range — user-configurable
  VLAN 1002–1005 → Reserved (legacy Token Ring/FDDI)
  VLAN 1006–4094 →  Extended range — requires VTP transparent mode
```

### 3.2 Access Ports
Connects to **end devices** (PCs, printers, IP phones, servers).
- Carries traffic for **ONE VLAN only**
- Strips 802.1Q tag before sending to device
- End device is unaware of VLAN membership

```
Switch config example (Cisco):
  interface FastEthernet0/1
   switchport mode access
   switchport access vlan 10
```

### 3.3 Trunk Ports
Connects **switch to switch** or **switch to router**.
- Carries traffic for **MULTIPLE VLANs** simultaneously
- Uses **802.1Q tagging** to identify which frame belongs to which VLAN
- Essential for VLAN traffic to span multiple switches

```
Switch config example (Cisco):
  interface GigabitEthernet0/1
   switchport mode trunk
   switchport trunk allowed vlan 10,20,30
   switchport trunk native vlan 999
```

### 3.4 802.1Q Tagging
The IEEE standard for VLAN tagging. Inserts a **4-byte tag** into the Ethernet frame:

```
Standard Ethernet frame:
[ Dest MAC | Src MAC | EtherType | Data | FCS ]

802.1Q tagged frame:
[ Dest MAC | Src MAC | 802.1Q Tag | EtherType | Data | FCS ]
                           │
                    4 bytes containing:
                    ├── TPID: 0x8100 (identifies as 802.1Q)
                    ├── PCP: Priority (QoS — 3 bits)
                    ├── DEI: Drop Eligible Indicator (1 bit)
                    └── VID: VLAN ID (12 bits — 0 to 4095)
```

### 3.5 Native VLAN
The VLAN assigned to **untagged traffic** on a trunk port. If a frame arrives on a trunk port without an 802.1Q tag, it is assigned to the native VLAN.

```
Default native VLAN = VLAN 1 (security risk — should be changed)
Best practice: Set native VLAN to an unused VLAN (e.g., VLAN 999)
```

**Security note:** Mismatched native VLANs between switches enable **VLAN hopping** attacks.

### 3.6 Voice VLAN
A separate VLAN dedicated to **VoIP traffic** (IP phones).

```
Benefits:
  ├── QoS prioritization — voice traffic gets low latency
  ├── Isolation from data traffic — no interference
  └── Security — voice separated from user data

Config: One port can carry both data VLAN and voice VLAN simultaneously
```

### 3.7 Management VLAN
Dedicated VLAN for accessing and managing network devices (switches, routers, APs) via SSH, HTTPS, or SNMP.

```
Best practice:
  ├── Separate from user VLANs
  ├── Highly restricted access (firewall + ACL)
  ├── Never VLAN 1 (default — too well known)
  └── Only accessible from admin workstations
```

---

## 4. Inter-VLAN Routing

VLANs are isolated — they cannot talk to each other without a Layer 3 device.

### 4.1 Router-on-a-Stick
One physical router interface divided into **sub-interfaces** — one per VLAN. Trunk port connects router to switch.

```
Router sub-interface config (Cisco):
  interface GigabitEthernet0/0.10
   encapsulation dot1Q 10
   ip address 192.168.10.1 255.255.255.0

  interface GigabitEthernet0/0.20
   encapsulation dot1Q 20
   ip address 192.168.20.1 255.255.255.0
```

```
Traffic flow: VLAN 10 device → switch → trunk → router sub-interface
              → routing decision → back down trunk → VLAN 20 device
```

**Limitation:** All inter-VLAN traffic passes through one physical link — can become a bottleneck.

### 4.2 Layer 3 Switch (Most common enterprise approach)
A switch with built-in routing capabilities. Creates **SVI (Switched Virtual Interface)** for each VLAN.

```
Layer 3 switch config:
  interface Vlan10
   ip address 192.168.10.1 255.255.255.0
  interface Vlan20
   ip address 192.168.20.1 255.255.255.0
  ip routing  ← enables routing between VLANs
```

**Advantage:** Wire-speed inter-VLAN routing — no external router needed.

---

## 5. VLAN Types Summary

| VLAN Type | Purpose |
|-----------|---------|
| **Data VLAN** | Carries regular user traffic |
| **Voice VLAN** | Dedicated to VoIP — QoS prioritized |
| **Management VLAN** | Network device administration |
| **Native VLAN** | Untagged traffic on trunk ports |
| **Black hole VLAN** | Unused VLAN where disabled ports are assigned |
| **Guest VLAN** | Internet-only access for visitors — no LAN access |

---

## 6. VTP — VLAN Trunking Protocol

Cisco proprietary protocol that propagates VLAN configuration across all switches automatically.

| VTP Mode | Behavior |
|----------|---------|
| **Server** | Creates, modifies, deletes VLANs — propagates to others |
| **Client** | Receives VLAN info from server — cannot modify |
| **Transparent** | Doesn't participate in VTP — passes VTP messages, keeps local config |
| **Off** | VTP disabled entirely |

**Security risk:** A rogue switch in Server mode can overwrite all VLAN configs in the domain — always use VTP passwords and consider Transparent/Off mode.

---

## 7. Key Terms & Terminology

| Term | Meaning |
|------|---------|
| **VLAN** | Virtual LAN — logical network isolation on shared hardware |
| **VLAN ID** | Number (1–4094) identifying a VLAN |
| **Access port** | Switch port for end devices — one VLAN only |
| **Trunk port** | Switch port carrying multiple VLANs — 802.1Q tagged |
| **802.1Q** | IEEE VLAN tagging standard |
| **Native VLAN** | VLAN for untagged traffic on trunk ports |
| **Inter-VLAN routing** | Routing traffic between VLANs via Layer 3 |
| **SVI** | Switched Virtual Interface — Layer 3 interface for a VLAN on a Layer 3 switch |
| **Router-on-a-Stick** | One router interface with sub-interfaces for inter-VLAN routing |
| **VTP** | VLAN Trunking Protocol — Cisco proprietary VLAN propagation |
| **QoS** | Quality of Service — prioritizing certain traffic types (e.g., voice) |
| **Broadcast domain** | All devices that receive a broadcast — VLANs create separate broadcast domains |
| **VLAN hopping** | Attack that allows escaping one VLAN to access another |

---

## 8. 🔐 SOC Analyst Relevance

VLANs are a **core defense mechanism** — and a key area SOC analysts must understand deeply.

### VLAN-Based Attack Techniques

#### VLAN Hopping — Double Tagging Attack
```
Normal frame on trunk:  [ 802.1Q Tag VLAN 10 | Data ]

Double-tagged frame:    [ 802.1Q Tag VLAN 1 (native) | 802.1Q Tag VLAN 20 | Data ]

What happens:
  Switch 1: strips outer tag (native VLAN 1) → forwards on trunk
  Switch 2: reads inner tag (VLAN 20) → delivers to VLAN 20

Attacker in VLAN 10 reaches VLAN 20 without authorization!

Prevention: Change native VLAN from default (1) to unused VLAN
            Tag the native VLAN explicitly on all trunk ports
```

#### Switch Spoofing
Attacker configures their device to act as a switch and negotiate a trunk link using DTP (Dynamic Trunking Protocol) — gaining access to all VLANs.

```
Prevention: Disable DTP on all access ports
  switchport mode access         ← hard-set to access mode
  switchport nonegotiate         ← disable DTP negotiation
```

### SOC Monitoring for VLAN Attacks

| Indicator | What it suggests | Action |
|-----------|-----------------|--------|
| **Traffic from VLAN 1** | Native VLAN in use — hopping risk | Audit native VLAN config |
| **Double-tagged 802.1Q frames** | VLAN hopping attempt | Block, investigate source |
| **Unexpected trunk negotiation** | Switch spoofing attempt | Disable DTP, check port config |
| **Cross-VLAN traffic without router log** | Misconfiguration or bypass | Firewall and switch audit |
| **Guest VLAN reaching corporate VLAN** | Policy violation or attack | Immediate isolation |
| **New VLAN appearing in VTP domain** | Rogue switch or misconfiguration | VTP audit, check switch inventory |

### Network Segmentation Best Practices (SOC Perspective)

```
Recommended VLAN structure for a secure network:

VLAN 10  →  Management (switches, routers, APs — SSH/HTTPS only)
VLAN 20  →  Servers (database, file, application servers)
VLAN 30  →  User workstations (standard employees)
VLAN 40  →  Voice (VoIP phones — QoS enabled)
VLAN 50  →  DMZ (internet-facing services)
VLAN 60  →  Guest Wi-Fi (internet only — isolated from all others)
VLAN 70  →  IoT devices (cameras, printers — highly restricted)
VLAN 80  →  Security tools (SIEM, IDS sensors — read-only access)
VLAN 99  →  Black hole (unused ports assigned here — no routing)
VLAN 999 →  Native VLAN (unused — just for untagged trunk traffic)
```

### SIEM Queries for VLAN Security

```
# Detect traffic crossing from guest VLAN to corporate
index=firewall src_vlan=60 dst_vlan IN (20,30)

# Alert on double-tagged 802.1Q frames (VLAN hopping)
index=network 802.1q_tags_count=2

# Detect trunk negotiation on access ports (switch spoofing)
index=switch_logs event="trunk negotiation" port_type=access

# New VLAN added to VTP domain (unauthorized change)
index=switch_logs event="VLAN created" | alert on unexpected VLAN IDs

# Management VLAN accessed from non-admin subnet
index=network dst_vlan=10 NOT src_ip IN (admin_ip_list)
```

> **Real SOC scenario:** IDS alert fires on a double-tagged 802.1Q frame originating from a user workstation in VLAN 30. The inner tag targets VLAN 20 (server VLAN). This is a classic VLAN hopping attempt. SOC analyst immediately isolates the port, checks switch config to confirm native VLAN misconfiguration, remediates the config, and investigates the workstation for signs of compromise.

---

## 9. Common Interview Questions

**Q1. What is a VLAN and why is it used?**
> A VLAN is a logical grouping of network devices on shared physical switch infrastructure that behave as isolated networks. It's used for three main reasons: security (isolating departments limits breach impact), performance (reducing broadcast domains cuts unnecessary traffic), and flexibility (grouping devices logically without rewiring).

**Q2. What is the difference between an access port and a trunk port?**
> An access port connects to end devices and carries traffic for a single VLAN — the device is unaware of the VLAN. A trunk port connects switches together or to routers and carries traffic for multiple VLANs simultaneously using 802.1Q tags to identify which frame belongs to which VLAN.

**Q3. What is the native VLAN and why is it a security concern?**
> The native VLAN is the VLAN assigned to untagged traffic on a trunk port — VLAN 1 by default. It's a security concern because double-tagging VLAN hopping attacks exploit it — an attacker sends a double-tagged frame where the outer tag matches the native VLAN (stripped at first switch) revealing an inner tag for a restricted VLAN. Best practice: change native VLAN to an unused VLAN and tag it explicitly.

**Q4. What is VLAN hopping and how do you prevent it?**
> VLAN hopping allows an attacker to send traffic from one VLAN into another without authorization. Two methods: double-tagging (exploiting native VLAN) and switch spoofing (negotiating a trunk link via DTP). Prevention: change native VLAN from VLAN 1, disable DTP on all access ports, hard-set ports to access mode, and use explicit trunk VLAN allowlists.

**Q5. What is inter-VLAN routing and how is it implemented?**
> Inter-VLAN routing allows traffic to flow between VLANs through a Layer 3 device. It's implemented two ways: Router-on-a-Stick (one router physical interface with sub-interfaces per VLAN connected via trunk) or a Layer 3 switch (creates SVIs for each VLAN and routes internally at wire speed — preferred for enterprise).

**Q6. How does VLAN segmentation help limit the impact of ransomware?**
> If a user workstation is infected with ransomware, proper VLAN segmentation limits it to the user VLAN — it cannot directly reach the server VLAN, backup systems, or management VLAN without passing through a router with ACL rules. This prevents lateral movement and mass encryption of network shares, giving the SOC team time to isolate the infected subnet.

---

## 10. Quick Revision Summary

```
VLAN function       →  Logical network isolation on shared switch hardware
VLAN ID range       →  1–4094 (VLAN 1 = default, avoid using)

Port types:
  Access port       →  One VLAN, for end devices, no tag visible to device
  Trunk port        →  Multiple VLANs, 802.1Q tagged, switch-to-switch/router

802.1Q tag          →  4-byte field added to frame on trunk ports
Native VLAN         →  VLAN for untagged trunk traffic (change from VLAN 1!)

Inter-VLAN routing:
  Router-on-a-Stick →  Sub-interfaces on one router port
  Layer 3 switch    →  SVIs — preferred enterprise method

VLAN types:
  Data              →  User traffic
  Voice             →  VoIP — QoS prioritized
  Management        →  Network device admin — highly restricted
  Guest             →  Internet only — no LAN access
  Black hole (99)   →  Unused ports — no routing

Key attacks:
  Double tagging    →  Exploit native VLAN → hop to restricted VLAN
  Switch spoofing   →  Fake switch negotiates trunk → access all VLANs

Prevention:
  Change native VLAN from 1
  Disable DTP on access ports (switchport nonegotiate)
  Hard-set access ports (switchport mode access)
  Restrict trunk allowed VLANs explicitly

SOC focus:
  Watch for         →  Cross-VLAN traffic, double-tagged frames,
                        trunk negotiation on access ports
  Key defense       →  Proper segmentation limits lateral movement
```

---

*Previous topic → [Subnetting](../05-Subnetting/notes.md)*
*Next topic → [DNS & DHCP](../04-Protocols/dns-dhcp.md)*
