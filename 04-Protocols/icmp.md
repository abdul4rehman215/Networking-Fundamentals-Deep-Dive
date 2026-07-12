# 📡 ICMP — Full Deep Dive

> **Source:** Professor Messer | **Topic #:** 19 | **Module:** 2 | **Status:** ✅ Complete

---

## 1. Definition

**ICMP (Internet Control Message Protocol)** is a Layer 3 (Network Layer) protocol used by network devices to send error messages and operational information about network conditions. Unlike TCP and UDP, ICMP is not used to exchange data between systems — it is purely a diagnostic and reporting protocol.

**Key facts:**
- Protocol number: **1**
- No ports — ICMP does not use TCP or UDP port numbers
- No connection establishment — connectionless
- Encapsulated directly inside IP packets
- Defined in **RFC 792**

> ICMP is the network's way of talking to itself — reporting errors, testing connectivity, and providing diagnostic feedback between devices.

---

## 2. ICMP vs Ping — The Critical Difference

This is one of the most commonly misunderstood concepts:

```
ICMP  =  The full protocol
          ├── Many message types (0, 3, 5, 8, 11, 13, 14...)
          ├── Error reporting
          ├── Diagnostics
          └── Network control messages

Ping  =  ONE tool that uses just TWO ICMP message types:
          ├── Type 8  — Echo Request  (sent by ping)
          └── Type 0  — Echo Reply    (response from target)
```

**Analogy:**
> ICMP is like the entire postal system. Ping is like sending one specific type of letter — a "are you there?" card — and waiting for a reply.

---

## 3. ICMP Message Types

### Complete Reference Table

| Type | Name | Direction | Purpose |
|------|------|-----------|---------|
| **0** | Echo Reply | Response | Reply to ping (Type 8) |
| **3** | Destination Unreachable | Error | Host/network/port not reachable |
| **4** | Source Quench | Control | Deprecated — congestion control |
| **5** | Redirect | Control | Better route available (exploitable!) |
| **8** | Echo Request | Request | Ping — test if host is alive |
| **9** | Router Advertisement | Info | Router announces itself |
| **10** | Router Solicitation | Request | Host asks for router |
| **11** | Time Exceeded | Error | TTL expired in transit (traceroute) |
| **12** | Parameter Problem | Error | Bad IP header |
| **13** | Timestamp Request | Request | Time synchronization |
| **14** | Timestamp Reply | Response | Reply to timestamp request |
| **30** | Traceroute | Info | Deprecated traceroute info |

### Type 3 — Destination Unreachable Codes

| Code | Meaning |
|------|---------|
| 0 | Network unreachable |
| 1 | Host unreachable |
| 2 | Protocol unreachable |
| 3 | Port unreachable |
| 4 | Fragmentation needed but DF bit set |
| 5 | Source route failed |
| 13 | Communication administratively prohibited (firewall blocked) |

---

## 4. Ping — Deep Dive

### 4.1 How Ping Works

```
Step 1 — Source sends ICMP Type 8 (Echo Request) to destination IP
Step 2 — Destination receives the request
Step 3 — Destination sends ICMP Type 0 (Echo Reply) back to source
Step 4 — Source calculates Round Trip Time (RTT)
Step 5 — Results displayed: RTT in ms, packet loss %, TTL value

Example output:
  ping 8.8.8.8
  PING 8.8.8.8: 56 data bytes
  64 bytes from 8.8.8.8: icmp_seq=0 ttl=118 time=12.4 ms
  64 bytes from 8.8.8.8: icmp_seq=1 ttl=118 time=11.8 ms
  64 bytes from 8.8.8.8: icmp_seq=2 ttl=118 time=12.1 ms

  --- 8.8.8.8 ping statistics ---
  3 packets transmitted, 3 received, 0% packet loss
  round-trip min/avg/max = 11.8/12.1/12.4 ms
```

### 4.2 Reading Ping Output

| Field | Meaning |
|-------|---------|
| **bytes** | Size of ICMP packet (default 56/64 bytes) |
| **icmp_seq** | Sequence number — detect out-of-order or lost packets |
| **ttl** | Time To Live — remaining hops when packet arrived |
| **time** | Round Trip Time in milliseconds |
| **packet loss** | % of packets that didn't get a reply |

### 4.3 Estimating OS from TTL

```
TTL value received tells you approximate hops and likely OS:

TTL 128  →  Windows (starts at 128)
TTL 64   →  Linux / macOS (starts at 64)
TTL 255  →  Cisco routers / network devices
TTL 254  →  Solaris / AIX

Note: TTL decrements by 1 at each router hop
So TTL=118 from 8.8.8.8 = started at 128 (Windows) - 10 hops away
```

### 4.4 Ping Command Options

```bash
# Basic ping
ping 8.8.8.8

# Ping with specific count (Linux/Mac)
ping -c 4 8.8.8.8

# Ping with specific count (Windows)
ping -n 4 8.8.8.8

# Ping with larger packet size (test MTU)
ping -s 1400 8.8.8.8

# Continuous ping (Windows)
ping -t 8.8.8.8

# Ping with specific TTL
ping -ttl 10 8.8.8.8

# Ping IPv6
ping6 ::1
```

---

## 5. Traceroute — Deep Dive

### 5.1 How Traceroute Works

Traceroute maps every router hop between source and destination using TTL manipulation and ICMP Type 11 (Time Exceeded).

```
Step 1 — Send packet with TTL=1
          → First router decrements TTL to 0
          → Router drops packet, sends ICMP Type 11 back
          → Source records router 1's IP and RTT

Step 2 — Send packet with TTL=2
          → First router: TTL becomes 1, forwards
          → Second router: TTL becomes 0, sends Type 11 back
          → Source records router 2's IP and RTT

Step 3 — Repeat, incrementing TTL each time
          → Until packet reaches destination
          → Destination responds with ICMP Type 0 (Echo Reply)
          → Full path mapped

Example output:
  traceroute 8.8.8.8
   1  192.168.1.1     1.2 ms   (home router)
   2  10.0.0.1        5.4 ms   (ISP gateway)
   3  203.0.113.1     8.1 ms   (ISP core)
   4  72.14.215.1    11.3 ms   (Google edge)
   5  8.8.8.8        12.4 ms   (destination)
```

### 5.2 Traceroute Commands

```bash
# Linux/Mac
traceroute 8.8.8.8

# Windows
tracert 8.8.8.8

# Traceroute using TCP (bypasses ICMP blocks)
tcptraceroute 8.8.8.8

# MTR — combines ping + traceroute (real-time)
mtr 8.8.8.8
```

### 5.3 Reading Traceroute Output

```
Hop  RTT        IP/Hostname      Meaning
1    1.2 ms     192.168.1.1     Local router — fast
2    5.4 ms     10.0.0.1        ISP gateway
3    * * *      (no response)   Router blocking ICMP — firewall
4    200 ms     203.0.113.1     High latency — possible congestion
5    12.4 ms    8.8.8.8         Destination reached

* * * = router not responding to ICMP (common — not always a problem)
High RTT at specific hop = congestion or routing issue at that point
```

---

## 6. Ping vs Traceroute vs Pathping

| Tool | ICMP Types used | Purpose | OS |
|------|----------------|---------|-----|
| **ping** | Type 8 + Type 0 | Test host reachability + latency | All |
| **traceroute** | Type 8 + Type 11 | Map full path to destination | Linux/Mac |
| **tracert** | Type 8 + Type 11 | Same as traceroute | Windows |
| **pathping** | Type 8 + Type 11 | Combines ping + traceroute + packet loss per hop | Windows |
| **mtr** | Type 8 + Type 11 | Real-time continuous traceroute | Linux/Mac |

---

## 7. Key Terms & Terminology

| Term | Meaning |
|------|---------|
| **ICMP** | Internet Control Message Protocol — network diagnostics |
| **Echo Request** | ICMP Type 8 — "are you there?" sent by ping |
| **Echo Reply** | ICMP Type 0 — "yes I'm here" response |
| **TTL** | Time To Live — decremented at each hop, prevents loops |
| **Time Exceeded** | ICMP Type 11 — TTL reached 0, used by traceroute |
| **Destination Unreachable** | ICMP Type 3 — host/port/network not reachable |
| **RTT** | Round Trip Time — time for packet to reach destination and return |
| **Hop** | Each router a packet passes through |
| **ICMP tunneling** | Hiding data inside ICMP packets to bypass firewalls |
| **Smurf attack** | DDoS using ICMP broadcast amplification |
| **Ping flood** | Overwhelming target with rapid ICMP Echo Requests |
| **ICMP redirect** | Type 5 — tells host a better route exists (exploitable) |
| **Pathping** | Windows tool combining ping + traceroute |
| **MTR** | My Traceroute — real-time continuous path analysis |

---

## 8. 🔐 SOC Analyst Relevance

ICMP is simple but heavily abused. Here's what SOC analysts watch for:

### ICMP-Based Attacks

#### ICMP Tunneling
```
Normal ping packet:
  ICMP Type 8 | Payload: 56 bytes of zeros (padding)

Tunneled packet:
  ICMP Type 8 | Payload: "GET /secret HTTP/1.1\r\nHost: c2.evil.com"
               (actual C2 commands or exfiltrated data hidden in payload)

Why it works:
  ├── Many firewalls allow ICMP outbound
  ├── Payload not inspected by simple firewalls
  └── Blends with legitimate ping traffic

Tools: icmptunnel, ptunnel, icmpsh

Detection:
  ├── ICMP packets larger than normal (>64 bytes payload)
  ├── High frequency of ICMP to same destination
  ├── ICMP payload contains readable strings/patterns
  └── ICMP traffic to external IPs at unusual times
```

#### Smurf Attack (DDoS)
```
Attacker spoofs victim's IP as source
Sends ICMP Echo Request to broadcast address
ALL hosts on network reply to victim's IP simultaneously
Victim overwhelmed with ICMP replies

Prevention: Disable directed broadcast on routers
Detection: Massive ICMP volume from many sources to one destination
```

#### Ping Flood
```
Attacker sends rapid continuous ICMP Echo Requests
Overwhelms target's CPU/bandwidth processing replies
Simple but effective against unprotected targets

Detection: Abnormally high ICMP packet rate from single source
```

#### ICMP Redirect Attack (Type 5)
```
Attacker sends spoofed ICMP Type 5 (Redirect)
Tells victim "send traffic for X through MY router"
Victim reroutes traffic through attacker — MITM

Prevention: Disable ICMP redirect processing on hosts
Detection: Type 5 messages from unexpected sources
```

#### Ping of Death (Historical)
```
Sending oversized ICMP packet (>65,535 bytes)
Caused buffer overflow and system crash
Patched on all modern systems — historical only
```

### SOC Monitoring for ICMP

```
Normal ICMP:
  ├── Small payload (56-64 bytes)
  ├── Infrequent — admin troubleshooting only
  ├── Internal hosts pinging internal hosts
  └── Type 8 + Type 0 (ping) or Type 11 (traceroute)

Suspicious ICMP:
  ├── Large payload size (>100 bytes) → possible tunneling
  ├── High frequency from one source → flood attack
  ├── ICMP to external IPs at unusual hours → C2 tunneling
  ├── Type 5 (Redirect) from unknown source → redirect attack
  └── ICMP from many sources to one IP → Smurf DDoS
```

### SIEM Queries for ICMP Monitoring

```
# Detect oversized ICMP packets (tunneling)
index=network protocol=ICMP packet_size > 100
| stats count by src_ip, dst_ip | sort -count

# ICMP flood detection
index=network protocol=ICMP
| stats count by src_ip | where count > 1000

# ICMP to external IPs (possible C2 tunnel)
index=network protocol=ICMP dst_ip NOT IN (internal_ranges)
| stats count by src_ip, dst_ip

# ICMP redirect messages from unexpected sources
index=network icmp_type=5 NOT src_ip IN (known_routers)

# Traceroute detection (recon activity)
index=network icmp_type=8
| stats dc(dst_ip) as targets by src_ip | where targets > 10
```

---

## 9. Common Interview Questions

**Q1. What is the difference between ICMP and ping?**
> ICMP (Internet Control Message Protocol) is the full Layer 3 protocol with many message types used for error reporting, diagnostics, and network control. Ping is just one specific tool that uses two ICMP message types — Type 8 (Echo Request) to ask "are you there?" and Type 0 (Echo Reply) as the response. ICMP is the protocol; ping is one application of it.

**Q2. How does traceroute use ICMP?**
> Traceroute sends packets with incrementing TTL values starting at 1. Each router decrements TTL by 1 — when it reaches 0, the router drops the packet and sends back an ICMP Type 11 (Time Exceeded) message revealing its IP address. By incrementing TTL with each probe, traceroute maps every router hop to the destination.

**Q3. What is ICMP tunneling and how would you detect it?**
> ICMP tunneling hides data inside ICMP packet payloads — embedding C2 commands or exfiltrated data in what appears to be normal ping traffic. Detection: look for ICMP packets with payloads larger than the standard 56-64 bytes, high frequency ICMP to the same external destination, ICMP traffic at unusual times, and readable strings in packet payloads via DPI.

**Q4. What does TTL tell you in a ping response?**
> TTL (Time To Live) starts at the OS default (128 for Windows, 64 for Linux) and decrements by 1 at each router hop. The TTL value in a ping reply tells you approximately how many hops away the destination is and can help identify the destination's OS — TTL near 128 suggests Windows, near 64 suggests Linux/macOS.

**Q5. What is a Smurf attack?**
> A Smurf attack is a DDoS amplification attack where the attacker spoofs the victim's IP address and sends ICMP Echo Requests to a network's broadcast address. Every host on that network replies to the victim's IP simultaneously, overwhelming it with ICMP replies. Prevented by disabling directed broadcast forwarding on routers.

**Q6. Why would you see * * * in traceroute output?**
> Asterisks in traceroute indicate a router that is not responding to ICMP — either because it has ICMP blocked by firewall rules, is configured not to respond to TTL-exceeded messages, or the packet was lost. It doesn't necessarily mean the path is broken — subsequent hops may still respond normally.

---

## 10. Quick Revision Summary

```
ICMP:
  Layer        →  3 (Network Layer)
  Protocol #   →  1
  Purpose      →  Error reporting, diagnostics, control
  No ports     →  Doesn't use TCP/UDP port numbers

Key message types:
  Type 0   →  Echo Reply (ping response)
  Type 3   →  Destination Unreachable
  Type 5   →  Redirect (exploitable)
  Type 8   →  Echo Request (ping)
  Type 11  →  Time Exceeded (traceroute)

Ping vs Traceroute:
  Ping        →  Type 8 + Type 0 → test reachability + RTT
  Traceroute  →  TTL manipulation + Type 11 → map full path

TTL OS fingerprinting:
  128  →  Windows
  64   →  Linux / macOS
  255  →  Cisco / network devices

ICMP attacks:
  Tunneling   →  Data hidden in ICMP payload → C2/exfil
  Smurf       →  Broadcast amplification DDoS
  Ping flood  →  Rapid Echo Requests → overwhelm target
  Redirect    →  Type 5 spoofed → reroute traffic (MITM)

SOC detection:
  Large ICMP payload  →  possible tunneling
  High ICMP volume    →  flood attack
  External ICMP       →  possible C2 tunnel
  Type 5 from unknown →  redirect attack
```

---

*Previous topic → [Network Protocols — HTTP, FTP, SSH](./network-protocols.md)*
*Part of Module 2 — Core Networking Concepts*
