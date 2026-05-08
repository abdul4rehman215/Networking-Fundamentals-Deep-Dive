# 🌐 TCP/IP Model — Full Deep Dive

> **Source:** Professor Messer | **Topic #:** 09 | **Module:** 2 | **Status:** ✅ Complete

---

## 1. Definition

The **TCP/IP Model** (Transmission Control Protocol / Internet Protocol) is the practical networking framework that the modern internet actually runs on. Developed by the U.S. Department of Defense in the 1970s, it defines how data is packaged, addressed, transmitted, routed, and received across interconnected networks.

Unlike the OSI Model (7 layers — a theoretical reference), the TCP/IP Model has **4 layers** and directly maps to real protocols in use today.

> If the OSI Model is the architect's blueprint, TCP/IP is the actual building. OSI tells you *how* to think about networking. TCP/IP is *what* your traffic actually does.

---

## 2. The 4 Layers — Overview

```
┌─────────────────────────────────┐
│     Layer 4 — Application       │  HTTP, HTTPS, DNS, FTP, SMTP, SSH
├─────────────────────────────────┤
│     Layer 3 — Transport         │  TCP, UDP
├─────────────────────────────────┤
│     Layer 2 — Internet          │  IP (IPv4, IPv6), ICMP, ARP
├─────────────────────────────────┤
│     Layer 1 — Network Access    │  Ethernet, Wi-Fi, MAC, Cables
└─────────────────────────────────┘
```

---

## 3. How It Works — Step by Step

When you load a webpage (e.g., https://google.com):

```
Application Layer  →  Browser creates HTTP GET request
                       DNS resolves google.com → IP address

Transport Layer    →  TCP breaks request into segments
                       Assigns source/destination port numbers
                       Performs 3-way handshake (SYN → SYN-ACK → ACK)

Internet Layer     →  IP header added: source IP + destination IP
                       Router selects best path across the internet

Network Access     →  Frame created with source/destination MAC
                       Bits transmitted as electrical/optical/radio signals

                    ↓ (travels across the internet)

Network Access     →  Bits received, frame extracted
Internet Layer     →  IP header checked, packet routed
Transport Layer    →  Segments reassembled in correct order
Application Layer  →  HTTP response rendered by browser — page loads
```

---

## 4. The 4 Layers — Deep Dive

### Layer 4 — Application Layer
**OSI equivalent:** Layers 5 (Session) + 6 (Presentation) + 7 (Application)

This is where all user-facing network services live. Applications use this layer to communicate over the network.

| Protocol | Port | Purpose |
|----------|------|---------|
| **HTTP** | 80 | Web traffic (unencrypted) |
| **HTTPS** | 443 | Web traffic (encrypted via TLS) |
| **DNS** | 53 | Domain name to IP resolution |
| **FTP** | 20/21 | File transfer |
| **SFTP** | 22 | Secure file transfer |
| **SSH** | 22 | Secure remote shell access |
| **SMTP** | 25 | Sending email |
| **IMAP** | 143 | Receiving email (synced) |
| **POP3** | 110 | Receiving email (downloaded) |
| **DHCP** | 67/68 | Automatic IP assignment |
| **SNMP** | 161 | Network device monitoring |
| **RDP** | 3389 | Remote desktop (Windows) |
| **Telnet** | 23 | Remote shell (insecure — avoid) |
| **NTP** | 123 | Network time synchronization |

### Layer 3 — Transport Layer
**OSI equivalent:** Layer 4 (Transport)

Responsible for **end-to-end communication** between applications. Uses port numbers to identify which application the data belongs to.

#### TCP — Transmission Control Protocol
Connection-oriented, reliable, ordered delivery.

**3-Way Handshake:**
```
Client                    Server
  │── SYN ──────────────► │   "I want to connect"
  │◄── SYN-ACK ───────────│   "OK, I acknowledge"
  │── ACK ──────────────► │   "Great, connection established"
  │                        │
  │══ Data transfer ══════ │
  │                        │
  │── FIN ──────────────► │   "I want to close"
  │◄── FIN-ACK ───────────│   "Closing acknowledged"
```

**TCP features:**
- Sequencing — packets arrive in correct order
- Acknowledgment — receiver confirms each segment
- Retransmission — lost segments are resent
- Flow control — prevents sender from overwhelming receiver
- Congestion control — adapts to network conditions

#### UDP — User Datagram Protocol
Connectionless, fast, no guaranteed delivery.

```
Client sends data → Server
No handshake. No acknowledgment. No retransmission.
If packets are lost → they stay lost.
```

#### TCP vs UDP Comparison

| Feature | TCP | UDP |
|---------|-----|-----|
| Connection | Connection-oriented | Connectionless |
| Reliability | Guaranteed delivery | Best effort |
| Ordering | In-order delivery | No ordering |
| Speed | Slower (overhead) | Faster (minimal overhead) |
| Header size | 20 bytes | 8 bytes |
| Use cases | Web, email, file transfer | Streaming, VoIP, DNS, gaming |

#### Well-Known Port Ranges
```
0 – 1023      →  Well-known ports (HTTP=80, HTTPS=443, SSH=22)
1024 – 49151  →  Registered ports (application-specific)
49152 – 65535 →  Dynamic/ephemeral ports (client-side, temporary)
```

### Layer 2 — Internet Layer
**OSI equivalent:** Layer 3 (Network)

Responsible for **logical addressing and routing** — getting packets from source to destination across multiple networks.

#### Key Protocols

| Protocol | Purpose |
|----------|---------|
| **IPv4** | 32-bit addressing — 4.3 billion addresses |
| **IPv6** | 128-bit addressing — virtually unlimited addresses |
| **ICMP** | Error reporting and diagnostics (ping, traceroute) |
| **ARP** | Resolves IP address → MAC address on local network |
| **RARP** | Reverse ARP — resolves MAC → IP (legacy) |
| **IGMP** | Manages multicast group membership |

#### IPv4 vs IPv6

| Feature | IPv4 | IPv6 |
|---------|------|------|
| Address length | 32-bit | 128-bit |
| Format | 192.168.1.1 | 2001:0db8:85a3::8a2e:0370:7334 |
| Total addresses | ~4.3 billion | 340 undecillion |
| Header size | 20 bytes | 40 bytes |
| NAT required | Yes (address exhaustion) | No |
| Security | Optional (IPSec) | Built-in IPSec |
| Configuration | Manual or DHCP | Auto-configuration (SLAAC) |

#### ICMP — Important for SOC
```
ICMP Type 0  →  Echo Reply (ping response)
ICMP Type 3  →  Destination Unreachable
ICMP Type 5  →  Redirect (can be exploited)
ICMP Type 8  →  Echo Request (ping)
ICMP Type 11 →  Time Exceeded (traceroute uses this)
```

### Layer 1 — Network Access Layer
**OSI equivalent:** Layer 1 (Physical) + Layer 2 (Data Link)

Handles the **physical transmission** of data — how bits move across the actual medium and how devices on the same local network communicate.

**Responsibilities:**
- Physical transmission (cables, fiber, Wi-Fi signals)
- MAC addressing (hardware addresses)
- Frame creation and error detection (FCS)
- Media access control (who can transmit when)

**Technologies:**
```
Wired      →  Ethernet (IEEE 802.3) — Cat5e, Cat6, fiber
Wireless   →  Wi-Fi (IEEE 802.11) — 802.11a/b/g/n/ac/ax
Other      →  Bluetooth, DSL, PPP, Frame Relay (legacy)
```

---

## 5. TCP/IP vs OSI — Full Mapping

```
TCP/IP Model          OSI Model              Protocols
─────────────────────────────────────────────────────────
                    ┌─ Layer 7: Application ─┐
Application  ───── │  Layer 6: Presentation  │ HTTP, HTTPS, DNS,
                    └─ Layer 5: Session ─────┘ FTP, SSH, SMTP

Transport    ───── │  Layer 4: Transport    │  TCP, UDP

Internet     ───── │  Layer 3: Network      │  IP, ICMP, ARP

                    ┌─ Layer 2: Data Link ───┐
Network Access ─── │  Layer 1: Physical     │  Ethernet, Wi-Fi,
                    └────────────────────────┘ MAC addresses
```

---

## 6. Data Encapsulation in TCP/IP

As data moves down the layers, each layer adds its own header:

```
Application  →  [  DATA  ]
Transport    →  [ TCP HDR |  DATA  ]             ← segment
Internet     →  [ IP HDR | TCP HDR |  DATA  ]    ← packet
Network Acc  →  [ ETH HDR | IP HDR | TCP HDR | DATA | FCS ]  ← frame
                                                        ↓
                              Transmitted as bits (0s and 1s)
```

On the receiving end, each layer **strips** its header (decapsulation) until the raw data reaches the application.

---

## 7. Key Terms & Terminology

| Term | Meaning |
|------|---------|
| **TCP** | Transmission Control Protocol — reliable, connection-oriented |
| **UDP** | User Datagram Protocol — fast, connectionless |
| **IP** | Internet Protocol — logical addressing and routing |
| **ICMP** | Internet Control Message Protocol — diagnostics and errors |
| **ARP** | Address Resolution Protocol — IP to MAC mapping |
| **Port** | 16-bit number identifying a specific application/service |
| **Socket** | Combination of IP address + port (e.g., 192.168.1.1:443) |
| **Segment** | TCP/UDP data unit at Transport layer |
| **Packet** | IP data unit at Internet layer |
| **Frame** | Data unit at Network Access layer |
| **3-way handshake** | TCP connection setup: SYN → SYN-ACK → ACK |
| **4-way termination** | TCP connection teardown: FIN → ACK → FIN → ACK |
| **Ephemeral port** | Temporary client-side port (49152–65535) |
| **TTL** | Time to Live — decremented at each hop, prevents loops |
| **Encapsulation** | Adding headers as data moves down the stack |
| **Decapsulation** | Removing headers as data moves up the stack |

---

## 8. 🔐 SOC Analyst Relevance

TCP/IP is the **foundation of everything** a SOC analyst does. Every log, every alert, every packet capture references TCP/IP concepts.

### Attacks Mapped to TCP/IP Layers

| Layer | Attack | Description |
|-------|--------|-------------|
| **Application** | SQL injection, XSS, phishing, C2 over HTTP/S | Exploit application protocols |
| **Application** | DNS tunneling | Exfiltrate data inside DNS queries |
| **Transport** | SYN flood (DDoS) | Send massive SYN packets, exhaust server state table |
| **Transport** | Port scanning | Probe TCP/UDP ports to map services |
| **Transport** | Session hijacking | Steal TCP sequence numbers to take over session |
| **Internet** | IP spoofing | Fake source IP address |
| **Internet** | ICMP tunneling | Exfiltrate data inside ping packets |
| **Internet** | Smurf attack | Amplified DDoS using ICMP broadcast |
| **Network Access** | ARP poisoning | Fake ARP replies — Man-in-the-Middle |
| **Network Access** | MAC spoofing | Fake hardware address |

### SOC Analysis Using TCP/IP

```
Reading a firewall log entry:
2026-05-08 03:14:22 | SRC: 45.33.32.156:54231 | DST: 10.0.1.50:22 | TCP | DENY

SOC analyst reads:
  ├── Source IP: 45.33.32.156 (external — check threat intel)
  ├── Source Port: 54231 (ephemeral — client side)
  ├── Destination IP: 10.0.1.50 (internal server)
  ├── Destination Port: 22 (SSH — someone trying to SSH in)
  ├── Protocol: TCP
  ├── Action: DENY (firewall blocked it)
  └── Time: 3am — suspicious off-hours activity
```

### TCP Flags — Critical for SOC

```
SYN   →  Synchronize — initiate connection
ACK   →  Acknowledge — confirm receipt
FIN   →  Finish — close connection gracefully
RST   →  Reset — abruptly terminate connection
PSH   →  Push — send data immediately
URG   →  Urgent — prioritize this data

Suspicious flag combinations:
  SYN only (no ACK)   →  SYN scan (Nmap default)
  FIN only            →  FIN scan (stealth scan)
  SYN+FIN             →  Illegal — never happens normally
  No flags (NULL)     →  NULL scan (evasion technique)
  All flags set       →  XMAS scan (evasion technique)
```

### SIEM Queries Using TCP/IP Knowledge

```
# Detect SYN flood — high SYN count from one source
index=firewall tcp_flag=SYN | stats count by src_ip | where count > 1000

# Port scan detection — one source hitting many ports
index=firewall | stats dc(dst_port) as ports by src_ip | where ports > 20

# SSH brute force — repeated TCP 22 connection attempts
index=firewall dst_port=22 action=allowed | stats count by src_ip

# ICMP tunneling — unusually large ping packets
index=network protocol=ICMP packet_size > 100

# Detect Nmap SYN scan — SYN packets with no follow-up ACK
index=firewall tcp_flag=SYN NOT tcp_flag=ACK | stats count by src_ip
```

---

## 9. Common Interview Questions

**Q1. What is the difference between the TCP/IP model and the OSI model?**
> The OSI model is a 7-layer theoretical reference framework used for understanding and troubleshooting network communications. The TCP/IP model is a 4-layer practical framework that the internet actually implements. OSI separates Session, Presentation, and Application into three layers; TCP/IP combines them into one Application layer. Both are used in networking — OSI for conceptual discussion, TCP/IP for real protocol implementation.

**Q2. What is the TCP 3-way handshake and why does it matter in security?**
> The 3-way handshake establishes a TCP connection: client sends SYN, server responds SYN-ACK, client confirms with ACK. It matters in security because SYN flood attacks exploit this — attackers send massive SYN packets without completing the handshake, exhausting the server's connection table (half-open connections) and causing denial of service.

**Q3. What is the difference between TCP and UDP? Give security examples.**
> TCP is connection-oriented and guarantees reliable, ordered delivery — used for HTTP, SSH, FTP. UDP is connectionless with no delivery guarantees — used for DNS, VoIP, streaming. Security examples: SYN floods target TCP's handshake; DNS amplification DDoS exploits UDP's connectionless nature (no source verification).

**Q4. What does a port number tell you as a SOC analyst?**
> A port number identifies the application or service the traffic is intended for. Standard ports (like 443 for HTTPS, 22 for SSH) tell me what protocol should be running. Unusual ports — like a connection on port 4444 (common Metasploit default) or 1337 — are red flags worth investigating. Traffic on standard ports doesn't guarantee it's legitimate — attackers tunnel C2 over port 443.

**Q5. What is ARP and how is it exploited?**
> ARP (Address Resolution Protocol) resolves IP addresses to MAC addresses on a local network. It's exploited through ARP poisoning — an attacker sends fake ARP replies associating their MAC address with a legitimate IP, redirecting traffic through their machine (Man-in-the-Middle attack). ARP has no authentication, making it inherently vulnerable.

**Q6. What TCP flags would you look for when detecting a port scan?**
> Nmap's default SYN scan sends SYN packets without completing the handshake — I'd look for many SYN packets from one source to many destination ports with no corresponding ACK. Stealth scans use FIN, NULL (no flags), or XMAS (SYN+FIN+URG+PSH) packets — combinations that are illegal in normal traffic and indicate scanning activity.

---

## 10. Quick Revision Summary

```
TCP/IP has 4 layers (vs OSI's 7):
  Layer 4 — Application   →  HTTP, HTTPS, DNS, FTP, SSH, SMTP
  Layer 3 — Transport     →  TCP (reliable), UDP (fast)
  Layer 2 — Internet      →  IP, ICMP, ARP
  Layer 1 — Network Access →  Ethernet, Wi-Fi, MAC addresses

TCP vs UDP:
  TCP  →  Reliable, ordered, connection-oriented (3-way handshake)
  UDP  →  Fast, connectionless, no delivery guarantee

Port ranges:
  0–1023      →  Well-known (HTTP=80, HTTPS=443, SSH=22, DNS=53)
  1024–49151  →  Registered
  49152–65535 →  Dynamic/ephemeral (client ports)

TCP flags SOC must know:
  SYN only           →  Connection initiation / SYN scan
  SYN+FIN            →  Illegal — scanning/evasion
  NULL (no flags)    →  NULL scan
  All flags          →  XMAS scan

Key attacks per layer:
  Application  →  SQLi, XSS, DNS tunneling, C2 over HTTPS
  Transport    →  SYN flood, port scan, session hijack
  Internet     →  IP spoofing, ICMP tunneling, Smurf attack
  Net Access   →  ARP poisoning, MAC spoofing
```

---

*Previous topic → [NAS & SAN](../02-Devices/nas-san.md)*
*Next topic → [Subnetting](../05-Subnetting/notes.md)*
