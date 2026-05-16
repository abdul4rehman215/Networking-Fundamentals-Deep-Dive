# 🌍 DNS & DHCP — Full Deep Dive

> **Source:** Professor Messer | **Topic #:** 12 | **Module:** 2 | **Status:** ✅ Complete

---

## 1. DNS — Domain Name System

### 1.1 Definition
**DNS** is a hierarchical, distributed naming system that translates human-readable domain names (google.com) into machine-readable IP addresses (142.250.80.46). It is often called the "phone book of the internet."

> Without DNS, every user would need to memorize the IP address of every website — DNS makes the internet humanly usable.

**Port:** UDP/TCP 53
**Protocol layer:** Application (TCP/IP Layer 4)

---

### 1.2 DNS Resolution — Step by Step

```
User types: www.google.com

Step 1 — Browser checks its LOCAL CACHE
          └── Found? → Done. Use cached IP.
          └── Not found? → Continue

Step 2 — OS checks HOSTS FILE (/etc/hosts or C:\Windows\System32\drivers\etc\hosts)
          └── Found? → Done. Use entry.
          └── Not found? → Continue

Step 3 — Query sent to RECURSIVE RESOLVER (your ISP or 8.8.8.8)
          └── Resolver checks its cache
          └── Found? → Return IP to client
          └── Not found? → Continue

Step 4 — Resolver queries ROOT NAME SERVER (13 root server clusters)
          └── Root doesn't know the IP
          └── Root returns: "Ask the .com TLD server"

Step 5 — Resolver queries TLD SERVER (.com, .org, .net etc.)
          └── TLD doesn't know the IP
          └── TLD returns: "Ask Google's authoritative server"

Step 6 — Resolver queries AUTHORITATIVE NAME SERVER (Google's DNS)
          └── Authoritative server knows the answer
          └── Returns: www.google.com = 142.250.80.46

Step 7 — Resolver caches the result (TTL determines how long)
          └── Returns IP to client

Step 8 — Browser connects to 142.250.80.46
          └── Page loads
```

---

### 1.3 DNS Record Types

| Record | Purpose | Example |
|--------|---------|---------|
| **A** | Maps domain → IPv4 address | google.com → 142.250.80.46 |
| **AAAA** | Maps domain → IPv6 address | google.com → 2607:f8b0::... |
| **CNAME** | Alias — maps domain to another domain | www → google.com |
| **MX** | Mail server for the domain | google.com → aspmx.l.google.com |
| **NS** | Authoritative name servers for domain | google.com → ns1.google.com |
| **PTR** | Reverse lookup — IP → domain name | 142.250.80.46 → google.com |
| **TXT** | Text records — used for SPF, DKIM, DMARC | "v=spf1 include:..." |
| **SOA** | Start of Authority — zone info and primary NS | Serial, refresh, retry values |
| **SRV** | Service location records | _sip._tcp.example.com |
| **CAA** | Certificate Authority Authorization | Which CAs can issue certs for domain |

---

### 1.4 DNS Components

| Component | Role |
|-----------|------|
| **DNS Client (Resolver)** | The device making DNS queries |
| **Recursive Resolver** | ISP or public DNS (8.8.8.8, 1.1.1.1) — does the legwork |
| **Root Name Server** | Top of the DNS hierarchy — 13 logical root servers (a–m.root-servers.net) |
| **TLD Server** | Manages top-level domains (.com, .org, .uk, .pk) |
| **Authoritative Name Server** | Has the definitive answer for a specific domain |
| **DNS Cache** | Temporary storage of resolved queries (reduces load, speeds response) |
| **TTL (Time to Live)** | How long a DNS record is cached before being refreshed |

---

### 1.5 DNS Query Types

| Type | Description |
|------|-------------|
| **Recursive query** | Client asks resolver to do all the work and return final answer |
| **Iterative query** | Resolver asks each server in turn, gets referrals, does its own legwork |
| **Inverse/Reverse query** | Resolves IP → domain name (uses PTR records) |

---

### 1.6 Public DNS Servers

| Provider | Primary | Secondary |
|----------|---------|-----------|
| **Google** | 8.8.8.8 | 8.8.4.4 |
| **Cloudflare** | 1.1.1.1 | 1.0.0.1 |
| **OpenDNS** | 208.67.222.222 | 208.67.220.220 |
| **Quad9** | 9.9.9.9 | 149.112.112.112 |

---

## 2. DHCP — Dynamic Host Configuration Protocol

### 2.1 Definition
**DHCP** is a network protocol that automatically assigns IP configuration to devices when they connect to a network — eliminating the need for manual IP configuration.

**What DHCP assigns:**
```
├── IP address
├── Subnet mask
├── Default gateway (router IP)
├── DNS server address(es)
├── Lease duration
└── Optional: NTP server, TFTP server, domain name
```

**Ports:** UDP 67 (server) and UDP 68 (client)
**Protocol layer:** Application (TCP/IP Layer 4)

---

### 2.2 DHCP DORA Process — Step by Step

```
DORA = Discover → Offer → Request → Acknowledge

Step 1 — DISCOVER (client broadcasts)
  Client has no IP yet → broadcasts to 255.255.255.255
  "Is there a DHCP server? I need an IP address!"
  Source IP: 0.0.0.0  Destination: 255.255.255.255

Step 2 — OFFER (server unicasts/broadcasts)
  DHCP server receives broadcast
  "Here's an available IP: 192.168.1.50
   Subnet mask: 255.255.255.0
   Gateway: 192.168.1.1
   DNS: 8.8.8.8
   Lease: 24 hours"

Step 3 — REQUEST (client broadcasts)
  Client accepts the offer (if multiple servers offered, picks one)
  Broadcasts acceptance so other servers know
  "I'd like to use 192.168.1.50 from Server X"

Step 4 — ACKNOWLEDGE (server confirms)
  DHCP server confirms the lease
  "Confirmed. 192.168.1.50 is yours for 24 hours"
  Client configures its network interface with the assigned settings
```

---

### 2.3 DHCP Lease

A **lease** is the time period an IP address is assigned to a device. After expiry, the device must renew.

```
Lease renewal process:
  At 50% of lease time → client sends unicast REQUEST to DHCP server
  Server replies with ACK → lease renewed, timer resets

  At 87.5% of lease time → client broadcasts REQUEST (server may be unreachable)
  If no response → client tries until lease expires

  Lease expires → client loses IP, restarts DORA process
```

---

### 2.4 DHCP Reservation (Static DHCP)
Assigns the **same IP every time** to a specific device based on its MAC address.

```
Example: Always assign 192.168.1.10 to the printer with MAC AA:BB:CC:DD:EE:FF
Used for: servers, printers, network devices that need predictable IPs
Benefit: DHCP management + static IP behavior
```

---

### 2.5 DHCP Scope and Options

| Term | Meaning |
|------|---------|
| **Scope** | Pool of IP addresses DHCP can assign (e.g., 192.168.1.10–192.168.1.200) |
| **Exclusion** | IPs within scope reserved for static assignment (not handed out) |
| **Reservation** | Specific IP always assigned to specific MAC |
| **Lease time** | How long IP is assigned before renewal required |
| **Option 3** | Default gateway |
| **Option 6** | DNS server |
| **Option 15** | Domain name |
| **Option 43** | Vendor-specific info (used by VoIP phones, APs) |
| **Option 66/67** | TFTP server/filename — for PXE boot |

---

## 3. DNS & DHCP Together

```
Device joins network:

1. DHCP assigns: IP 192.168.1.50, Gateway 192.168.1.1, DNS 192.168.1.5
2. User opens browser → types google.com
3. DNS query sent to 192.168.1.5 (internal DNS server)
4. Internal DNS forwards to 8.8.8.8 (if not cached)
5. IP returned → browser connects
```

Many organizations run **internal DNS** alongside DHCP:
- Internal DNS resolves corporate hostnames (fileserver.company.local)
- DHCP registers device hostnames in DNS dynamically (DDNS)
- SOC can look up "which IP had hostname MUSFIRA-PC at 3am?"

---

## 4. Key Terms & Terminology

| Term | Meaning |
|------|---------|
| **DNS** | Domain Name System — translates domains to IPs |
| **DHCP** | Dynamic Host Configuration Protocol — auto IP assignment |
| **TTL** | Time to Live — DNS cache duration OR IP packet hop counter |
| **A record** | DNS record mapping domain to IPv4 |
| **PTR record** | Reverse DNS — IP to domain |
| **MX record** | Mail exchange server for a domain |
| **DORA** | Discover, Offer, Request, Acknowledge — DHCP process |
| **Lease** | Time period an IP is assigned to a device |
| **Scope** | Pool of IPs a DHCP server can assign |
| **Reservation** | Static IP via DHCP based on MAC address |
| **Recursive resolver** | DNS server doing all the work for the client |
| **Authoritative server** | DNS server with definitive answer for a domain |
| **DDNS** | Dynamic DNS — auto-updates DNS when DHCP assigns IP |
| **Split DNS** | Different DNS answers for internal vs external queries |
| **DNS forwarder** | DNS server passing unknown queries to another server |

---

## 5. 🔐 SOC Analyst Relevance

DNS and DHCP are among the **most critical data sources** in any SOC. Here's why:

### DNS — The Most Abused Protocol in Security

#### DNS Tunneling
Attacker encodes data inside DNS queries/responses to exfiltrate data or establish C2 channel through firewall.

```
Normal DNS query:  A record for google.com → 142.250.80.46

Tunneled query:    A record for
                   aGVsbG8gd29ybGQ=.evil-c2-domain.com
                   (base64 encoded data hidden in subdomain)

Detection signs:
  ├── Unusually long subdomain names
  ├── High volume of DNS queries to same domain
  ├── DNS queries with random-looking subdomains
  ├── Large DNS response sizes
  └── DNS queries at unusual times/intervals
```

#### DNS Hijacking
Attacker modifies DNS responses to redirect users to malicious sites.

```
Types:
  Local hijacking   →  Malware changes device's DNS settings
  Router hijacking  →  Attacker modifies router's DNS config
  DNS server attack →  Compromise DNS server directly
  BGP hijacking     →  Redirect DNS traffic at routing level
```

#### DNS Cache Poisoning
Attacker injects false DNS records into a resolver's cache — all users get the wrong IP.

```
Legitimate: example.com → 93.184.216.34
Poisoned:   example.com → 45.33.32.156 (attacker's server)
```

Prevention: **DNSSEC** (DNS Security Extensions) — cryptographically signs DNS records.

#### Fast-Flux DNS
Attackers rapidly rotate IP addresses for a single domain — makes C2 infrastructure hard to block.

```
Query 1 (10:00:01): evil.com → 1.2.3.4
Query 2 (10:00:02): evil.com → 5.6.7.8
Query 3 (10:00:03): evil.com → 9.10.11.12
TTL = 60 seconds — changes every minute
```

#### Domain Generation Algorithms (DGA)
Malware generates hundreds of random domain names daily — tries each until it finds an active C2 server.

```
Examples of DGA domains:
  xkq9mzpla2.com
  b7tnwq3mc.net
  yzx8vklpq.org
Detection: ML-based DNS analytics, high NXDOMAIN response rate
```

---

### DHCP — IP Address Forensics

DHCP logs are **critical for incident response** — they answer "which device had this IP at this time?"

```
DHCP log entry:
  2026-05-16 03:14:22 | ASSIGN | IP: 10.0.3.45 |
  MAC: AA:BB:CC:DD:EE:FF | Hostname: MUSFIRA-PC | Lease: 24h

SOC use case:
  Alert fires for malicious activity from 10.0.3.45 at 03:14
  → Pull DHCP log → "MUSFIRA-PC, MAC AA:BB:CC:DD:EE:FF"
  → Cross-reference with AD logs → identify user
  → Pull endpoint logs for MUSFIRA-PC → full investigation
```

#### DHCP Attacks

| Attack | Description | Detection |
|--------|-------------|-----------|
| **DHCP Starvation** | Attacker floods DHCP server with requests (fake MACs) — exhausts IP pool — legitimate devices can't get IPs | Spike in DHCP Discover messages, many short leases from different MACs |
| **Rogue DHCP Server** | Attacker runs unauthorized DHCP server — assigns fake gateways/DNS — MITM or DNS hijack | DHCP offers from unexpected source IPs/MACs |
| **DHCP Snooping bypass** | Attacker bypasses switch protection on ports | DHCP offers on untrusted ports |

**Prevention:**
- **DHCP Snooping** — switch feature that only allows DHCP offers from trusted ports
- **802.1X port authentication** — devices must authenticate before getting DHCP
- **Rate limiting** — limit DHCP requests per port

---

### SIEM Queries for DNS & DHCP

```
# DNS tunneling — long subdomain names
index=dns | eval subdomain_len=len(query) | where subdomain_len > 50

# High NXDOMAIN rate (possible DGA malware)
index=dns response_code=NXDOMAIN | stats count by src_ip | where count > 100

# Fast-flux detection — same domain returning many different IPs
index=dns | stats dc(answer_ip) as unique_ips by query | where unique_ips > 10

# DNS queries to newly registered domains
index=dns | lookup newly_registered_domains query OUTPUT is_new | where is_new=true

# Rogue DHCP server — DHCP offer from unknown server
index=dhcp message_type=OFFER NOT src_ip IN (authorized_dhcp_servers)

# DHCP starvation — flood of discover messages
index=dhcp message_type=DISCOVER | stats count by src_mac | where count > 50

# IP-to-hostname mapping for incident investigation
index=dhcp ip_address=10.0.3.45 earliest=-24h | table timestamp, hostname, mac_address
```

> **Real SOC scenario:** SIEM alert fires — a workstation is generating 500+ DNS queries per minute to random subdomains of the same domain. Subdomains contain long base64-encoded strings. This is classic DNS tunneling for C2 communication. SOC analyst immediately blocks the domain at the DNS firewall, isolates the host, pulls DHCP logs to confirm identity, and initiates malware analysis on the endpoint.

---

## 6. Common Interview Questions

**Q1. What is DNS and how does resolution work?**
> DNS translates domain names to IP addresses. Resolution: client queries recursive resolver → resolver checks cache → if not found, queries root server → root refers to TLD server → TLD refers to authoritative server → authoritative returns the IP → resolver caches it and returns to client. All typically under 100ms.

**Q2. What is the DHCP DORA process?**
> DORA stands for Discover, Offer, Request, Acknowledge. The client broadcasts a Discover message looking for a DHCP server. The server responds with an Offer containing available IP configuration. The client broadcasts a Request accepting the offer. The server sends an Acknowledge confirming the lease. The client then configures its interface with the assigned settings.

**Q3. What is DNS tunneling and how would you detect it?**
> DNS tunneling encodes data inside DNS queries/responses to bypass firewalls. Attackers use it for C2 communication or data exfiltration. Detection: look for unusually long subdomain names, high query volumes to the same domain, random-looking encoded subdomains, large DNS response sizes, and beaconing patterns in DNS logs.

**Q4. What is a rogue DHCP server and why is it dangerous?**
> A rogue DHCP server is an unauthorized DHCP server on the network. It's dangerous because it can hand out malicious gateway or DNS addresses — redirecting all traffic through an attacker's machine (MITM) or to attacker-controlled DNS servers. Prevented by DHCP Snooping — a switch feature that only permits DHCP offers from trusted ports.

**Q5. How do DHCP logs help in a SOC investigation?**
> DHCP logs map IP addresses to MAC addresses and hostnames at specific timestamps. When a security alert fires on an IP address, DHCP logs tell the SOC analyst exactly which physical device had that IP during the incident — enabling identification of the affected machine and user for further investigation.

**Q6. What is DNS cache poisoning and how is it prevented?**
> DNS cache poisoning inserts false records into a DNS resolver's cache — redirecting users to malicious sites when they query a legitimate domain. It's prevented by DNSSEC (DNS Security Extensions), which cryptographically signs DNS records so resolvers can verify their authenticity.

**Q7. What DNS record type would you check to investigate email spoofing?**
> TXT records — specifically SPF (Sender Policy Framework), DKIM (DomainKeys Identified Mail), and DMARC records. SPF defines which mail servers are authorized to send email for a domain, DKIM adds cryptographic signatures to emails, and DMARC tells receiving servers what to do when SPF/DKIM checks fail.

---

## 7. Quick Revision Summary

```
DNS:
  Function    →  Translates domain names to IP addresses
  Port        →  53 (UDP for queries, TCP for zone transfers)
  Record types:
    A          →  Domain → IPv4
    AAAA       →  Domain → IPv6
    CNAME      →  Alias to another domain
    MX         →  Mail server
    PTR        →  IP → Domain (reverse)
    TXT        →  SPF, DKIM, DMARC
  Resolution  →  Client → Resolver → Root → TLD → Authoritative

DHCP:
  Function    →  Auto-assigns IP config to devices
  Ports       →  67 (server), 68 (client) — UDP
  DORA:
    Discover  →  Client broadcasts "I need an IP"
    Offer     →  Server offers available IP + config
    Request   →  Client accepts the offer
    Acknowledge → Server confirms the lease

SOC key threats:
  DNS tunneling     →  Data exfil/C2 in DNS queries
  DNS hijacking     →  Redirect users to malicious sites
  Cache poisoning   →  False DNS records injected
  Fast-flux         →  Rapid IP rotation for C2
  DGA               →  Algorithmically generated C2 domains
  DHCP starvation   →  Exhaust IP pool — denial of service
  Rogue DHCP        →  Fake server assigns malicious gateway/DNS

Key SOC use:
  DNS logs    →  Detect C2 beaconing, tunneling, DGA, malicious domains
  DHCP logs   →  Map IP → MAC → hostname → user during investigation
```

---

*Previous topic → [VLANs](../03-TCP-IP/vlans.md)*
*Next topic → [Network Protocols — HTTP, FTP, SSH](./network-protocols.md)*
