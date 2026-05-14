# 🧮 Subnetting — Full Deep Dive

> **Source:** Professor Messer | **Topic #:** 10 | **Module:** 2 | **Status:** ✅ Complete

---

## 1. Definition

**Subnetting** is the process of dividing a single large IP network into multiple smaller sub-networks (subnets). Each subnet is an isolated logical segment of the larger network, with its own address range and broadcast domain.

**Core goals:**
- **Security** — isolate departments/systems from each other
- **Performance** — reduce broadcast traffic within each segment
- **Efficiency** — make better use of limited IPv4 address space
- **Control** — manage traffic flow between segments via routers

> Think of subnetting like dividing a large open office floor into separate rooms — people in each room communicate freely, but going between rooms requires passing through a door (router). Each room has its own space, and what happens in one room doesn't disturb the others.

---

## 2. IP Address Fundamentals

### IPv4 Address Structure
An IPv4 address is **32 bits** long, written as 4 octets in decimal:

```
192      .    168     .      1     .    100
11000000 . 10101000 . 00000001 . 01100100
  Octet 1    Octet 2    Octet 3    Octet 4
```

Every IPv4 address has two parts:
- **Network portion** — identifies which network the device belongs to
- **Host portion** — identifies the specific device within that network

The **subnet mask** determines where the network portion ends and the host portion begins.

### Binary Conversion Quick Reference
| Decimal | Binary |
|---------|--------|
| 0 | 00000000 |
| 128 | 10000000 |
| 192 | 11000000 |
| 224 | 11100000 |
| 240 | 11110000 |
| 248 | 11111000 |
| 252 | 11111100 |
| 254 | 11111110 |
| 255 | 11111111 |

---

## 3. Subnet Mask & CIDR Notation

### Subnet Mask
A 32-bit number that uses **1s** to mark the network portion and **0s** to mark the host portion:

```
IP address:    192.168.1.100   →  11000000.10101000.00000001.01100100
Subnet mask:   255.255.255.0   →  11111111.11111111.11111111.00000000
                                  |←── Network (24 bits) ───→|← Host →|
```

### CIDR Notation
**CIDR (Classless Inter-Domain Routing)** shortens the subnet mask into a prefix length — the number of 1-bits in the mask:

```
255.255.255.0   =  /24  (24 ones in the mask)
255.255.0.0     =  /16
255.0.0.0       =  /8
255.255.255.128 =  /25
255.255.255.252 =  /30
```

---

## 4. Classful Addressing (Legacy)

Before CIDR, IP addresses were divided into fixed classes:

| Class | Range | Default Mask | Hosts | Use |
|-------|-------|-------------|-------|-----|
| **A** | 1.0.0.0 – 126.255.255.255 | /8 (255.0.0.0) | 16,777,214 | Large organizations |
| **B** | 128.0.0.0 – 191.255.255.255 | /16 (255.255.0.0) | 65,534 | Medium organizations |
| **C** | 192.0.0.0 – 223.255.255.255 | /24 (255.255.255.0) | 254 | Small networks |
| **D** | 224.0.0.0 – 239.255.255.255 | N/A | N/A | Multicast |
| **E** | 240.0.0.0 – 255.255.255.255 | N/A | N/A | Reserved/Research |

> Classful addressing is obsolete — CIDR replaced it. But you'll still see Class A/B/C referenced in certifications and interviews.

---

## 5. Private IP Address Ranges

These ranges are reserved for internal/private use — NOT routable on the internet:

| Class | Private Range | CIDR | Common Use |
|-------|--------------|------|-----------|
| A | 10.0.0.0 – 10.255.255.255 | 10.0.0.0/8 | Large enterprise networks |
| B | 172.16.0.0 – 172.31.255.255 | 172.16.0.0/12 | Medium networks |
| C | 192.168.0.0 – 192.168.255.255 | 192.168.0.0/16 | Home/small office |

**Special addresses:**
```
127.0.0.0/8     →  Loopback (127.0.0.1 = localhost)
169.254.0.0/16  →  APIPA — auto-assigned when DHCP fails
0.0.0.0         →  Default route / "any address"
255.255.255.255 →  Limited broadcast
```

---

## 6. Subnetting Calculations

### Key Formulas
```
Number of hosts per subnet   =  2^(host bits) - 2
  (-2 for network address and broadcast address)

Number of subnets            =  2^(borrowed bits)

Host bits                    =  32 - prefix length
```

### CIDR Cheat Sheet

| CIDR | Subnet Mask | Host Bits | Usable Hosts | Typical Use |
|------|-------------|-----------|--------------|-------------|
| /8 | 255.0.0.0 | 24 | 16,777,214 | Very large networks |
| /16 | 255.255.0.0 | 16 | 65,534 | Large enterprise |
| /24 | 255.255.255.0 | 8 | 254 | Standard LAN segment |
| /25 | 255.255.255.128 | 7 | 126 | Split /24 into 2 |
| /26 | 255.255.255.192 | 6 | 62 | Small department |
| /27 | 255.255.255.224 | 5 | 30 | Small team |
| /28 | 255.255.255.240 | 4 | 14 | Very small segment |
| /29 | 255.255.255.248 | 3 | 6 | Small point-to-point |
| /30 | 255.255.255.252 | 2 | 2 | Point-to-point links (router-to-router) |
| /31 | 255.255.255.254 | 1 | 2 | Point-to-point (no broadcast — RFC 3021) |
| /32 | 255.255.255.255 | 0 | 1 | Single host route |

### Worked Example — Subnetting 192.168.1.0/24 into 4 subnets

**Step 1:** How many bits do we need to borrow to get 4 subnets?
```
2^n ≥ 4  →  2^2 = 4  →  borrow 2 bits
New prefix = /24 + 2 = /26
```

**Step 2:** How many hosts per subnet?
```
Host bits = 32 - 26 = 6
Hosts = 2^6 - 2 = 62 usable hosts per subnet
```

**Step 3:** What are the 4 subnets?
```
Block size = 256 - 192 = 64  (192 = value of /26 in last octet)

Subnet 1:  192.168.1.0/26    →  Hosts: .1 – .62    Broadcast: .63
Subnet 2:  192.168.1.64/26   →  Hosts: .65 – .126  Broadcast: .127
Subnet 3:  192.168.1.128/26  →  Hosts: .129 – .190 Broadcast: .191
Subnet 4:  192.168.1.192/26  →  Hosts: .193 – .254 Broadcast: .255
```

---

## 7. Special Addresses in a Subnet

Every subnet has three special addresses:

```
Network address   →  First address — identifies the subnet (NOT assignable)
Broadcast address →  Last address — sends to ALL hosts in subnet (NOT assignable)
Usable hosts      →  Everything in between

Example: 192.168.1.0/24
  Network:    192.168.1.0
  First host: 192.168.1.1
  Last host:  192.168.1.254
  Broadcast:  192.168.1.255
```

---

## 8. VLSM — Variable Length Subnet Masking

VLSM allows using **different subnet sizes** within the same network — allocating address space efficiently based on actual need.

```
Company needs:
  ├── HR dept:     50 hosts  →  /26 (62 usable)
  ├── IT dept:     25 hosts  →  /27 (30 usable)
  ├── Finance:     10 hosts  →  /28 (14 usable)
  └── Router link:  2 hosts  →  /30 (2 usable)

Without VLSM: all get /24 (254 hosts) → massive waste
With VLSM: each gets exactly what it needs → efficient
```

---

## 9. Key Terms & Terminology

| Term | Meaning |
|------|---------|
| **Subnet** | A subdivided segment of a larger network |
| **Subnet mask** | 32-bit value defining network vs host portions |
| **CIDR** | Classless Inter-Domain Routing — prefix length notation (e.g., /24) |
| **Network address** | First address in a subnet — identifies the subnet itself |
| **Broadcast address** | Last address in a subnet — sends to all hosts |
| **Usable hosts** | Total addresses minus 2 (network + broadcast) |
| **Default gateway** | Router IP that hosts use to reach other networks |
| **VLSM** | Variable Length Subnet Masking — different mask sizes in one network |
| **Supernetting** | Combining multiple smaller networks into a larger one (opposite of subnetting) |
| **APIPA** | 169.254.x.x — auto-assigned when DHCP unavailable |
| **Loopback** | 127.0.0.1 — refers to the local device itself |
| **Classful** | Old fixed-class system (A/B/C) — replaced by CIDR |
| **Block size** | 256 minus the subnet mask value — determines subnet increments |

---

## 10. 🔐 SOC Analyst Relevance

Subnetting knowledge is **used daily** in SOC work — here's exactly how:

### Identifying Traffic Context from IP Addresses

```
Alert: Src IP 10.10.5.45 → Dst IP 10.10.2.200

SOC analyst maps IPs to subnet assignments:
  10.10.5.0/24  →  User workstations (VLAN 50)
  10.10.2.0/24  →  Server zone (VLAN 20)

Conclusion: A workstation is talking directly to a server
  → Is this expected? What application?
  → Could be lateral movement after compromise
  → Check firewall rules — should this traffic be allowed?
```

### Subnet-Based Alert Triage

| Source Subnet | Destination Subnet | SOC Interpretation |
|---------------|-------------------|-------------------|
| User VLAN | Server VLAN | Possible lateral movement — investigate |
| User VLAN | Internet | Normal web traffic — check proxy logs |
| DMZ | Internal LAN | Red flag — DMZ should never initiate to internal |
| Guest VLAN | Corporate VLAN | Policy violation — should be blocked |
| Management VLAN | Any | Verify it's admin activity — very sensitive subnet |

### Network Segmentation as a Defense

```
Well-segmented network (SOC loves this):
  ├── 10.0.1.0/24  →  Management (firewalled, MFA required)
  ├── 10.0.2.0/24  →  Servers (DMZ-adjacent)
  ├── 10.0.3.0/24  →  User workstations
  ├── 10.0.4.0/24  →  Guest Wi-Fi (internet only, no LAN access)
  ├── 10.0.5.0/24  →  IoT devices (isolated)
  └── 10.0.6.0/24  →  Security tools (SIEM, IDS sensors)

If a workstation gets ransomware:
  → Blast radius limited to 10.0.3.0/24
  → Cannot reach servers in 10.0.2.0/24 without router
  → SOC can isolate just that subnet
```

### SIEM Queries Using Subnet Knowledge

```
# Detect traffic from guest subnet to corporate subnet
index=firewall src_ip=10.0.4.0/24 dst_ip=10.0.3.0/24

# Alert on any traffic from DMZ initiating to internal LAN
index=firewall src_ip=203.0.113.0/24 dst_ip=10.0.0.0/8 direction=inbound

# Find hosts using APIPA (DHCP failure — possible network issue)
index=network src_ip=169.254.0.0/16

# Detect loopback abuse (process communicating with itself — suspicious)
index=network dst_ip=127.0.0.0/8 NOT src_ip=127.0.0.0/8
```

> **Real SOC scenario:** Alert shows traffic from 10.0.4.55 (guest Wi-Fi) attempting connections to 10.0.2.100 (internal file server). Guest devices should have no route to the server subnet — this indicates either a misconfigured firewall rule or an attacker on the guest network trying to pivot internally. Immediate investigation and firewall rule review required.

---

## 11. Common Interview Questions

**Q1. What is subnetting and why is it used?**
> Subnetting divides a large network into smaller segments. It's used for three main reasons: security (isolating departments limits breach impact), performance (reducing broadcast domains reduces network congestion), and efficiency (making better use of limited IPv4 address space through CIDR).

**Q2. How many usable hosts does a /24 subnet have?**
> A /24 subnet has 8 host bits. 2^8 = 256 total addresses. Subtract 2 (network address and broadcast address) = **254 usable hosts**.

**Q3. What is the difference between a network address and a broadcast address?**
> The network address is the first address in a subnet — it identifies the subnet itself and cannot be assigned to a host. The broadcast address is the last address — any traffic sent to it reaches all hosts in the subnet. Neither can be assigned to a device.

**Q4. What is CIDR and why was it introduced?**
> CIDR (Classless Inter-Domain Routing) replaced the old classful system by allowing variable-length subnet masks. It was introduced to slow IPv4 address exhaustion and reduce the size of internet routing tables. Instead of fixed Class A/B/C boundaries, CIDR allows any prefix length (/8 through /32).

**Q5. How does subnetting help a SOC analyst?**
> Subnet knowledge lets a SOC analyst immediately contextualize any IP address — which department it belongs to, whether traffic between two subnets is expected, and whether firewall rules are being violated. It also underpins network segmentation — the key defense that limits lateral movement during a breach.

**Q6. What is VLSM and when would you use it?**
> VLSM (Variable Length Subnet Masking) allows using different subnet sizes within the same address space. You'd use it when different segments need different numbers of hosts — for example giving a 50-host department a /26, a 10-host department a /28, and a router link a /30, rather than wasting a /24 on each.

**Q7. What subnet does 192.168.10.130/26 belong to?**
> /26 means subnet mask 255.255.255.192, block size = 64.
> Subnets: .0, .64, .128, .192
> 130 falls in the .128 subnet.
> **Network: 192.168.10.128, Broadcast: 192.168.10.191, Hosts: .129–.190**

---

## 12. Quick Revision Summary

```
Subnetting        →  Dividing a large network into smaller segments
Subnet mask       →  Defines network vs host portions
CIDR notation     →  /prefix (e.g., /24 = 255.255.255.0)

Usable hosts      =  2^(host bits) - 2
Number of subnets =  2^(borrowed bits)
Host bits         =  32 - prefix length

Key subnets to memorize:
  /8   →  255.0.0.0       →  16,777,214 hosts
  /16  →  255.255.0.0     →  65,534 hosts
  /24  →  255.255.255.0   →  254 hosts
  /25  →  255.255.255.128 →  126 hosts
  /26  →  255.255.255.192 →  62 hosts
  /27  →  255.255.255.224 →  30 hosts
  /28  →  255.255.255.240 →  14 hosts
  /30  →  255.255.255.252 →  2 hosts (point-to-point)

Private ranges:
  10.0.0.0/8       →  Class A private
  172.16.0.0/12    →  Class B private
  192.168.0.0/16   →  Class C private

Special:
  127.0.0.1        →  Loopback (localhost)
  169.254.x.x      →  APIPA (DHCP failure)

SOC focus:
  Know your subnets →  Instantly contextualize any IP in an alert
  Segmentation      →  Limits lateral movement — core defense
  DMZ rule          →  DMZ should NEVER initiate connections to internal LAN
```

---

*Previous topic → [TCP/IP Model](../03-TCP-IP/notes.md)*
*Next topic → [VLANs](../03-TCP-IP/vlans.md)*
