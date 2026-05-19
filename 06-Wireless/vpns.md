# 🔒 VPNs — Full Deep Dive

> **Source:** Professor Messer | **Topic #:** 15 | **Module:** 3 | **Status:** ✅ Complete

---

## 1. Definition

A **VPN (Virtual Private Network)** creates an encrypted tunnel between a device or network and a remote network over an untrusted medium (usually the internet). It provides confidentiality, integrity, and authentication for traffic crossing public infrastructure.

**Core goals:**
- **Confidentiality** — encrypt traffic so it can't be read in transit
- **Integrity** — ensure traffic hasn't been tampered with
- **Authentication** — verify the identity of both ends of the tunnel
- **Privacy** — mask source IP from destination or third parties

> Think of a VPN as an armoured truck on a public road. Anyone can see the truck driving, but nobody can see what's inside or tamper with the cargo.

---

## 2. Types of VPNs

### 2.1 Site-to-Site VPN
Connects **two entire networks** together permanently over the internet.

```
Branch Office Network          Corporate HQ Network
  192.168.2.0/24                 192.168.1.0/24
       │                               │
  [Branch Router] ══ Encrypted ══ [HQ Firewall]
       │              Tunnel           │
  Branch devices see HQ network as if directly connected
```

**Characteristics:**
- Configured on routers/firewalls — transparent to end users
- Always-on tunnel — no user action required
- Uses IPSec in Tunnel mode typically
- Used for: multi-branch enterprises, data center connectivity

### 2.2 Remote Access VPN
Connects **individual users** to a corporate network from any location.

```
Home User (laptop)
       │
  [VPN Client] ══ Encrypted Tunnel ══ [VPN Gateway] ══ Corporate Network
       │
  User gets corporate IP — accesses internal resources as if in office
```

**Characteristics:**
- Requires VPN client software on user device
- User authenticates before tunnel established
- On-demand — user connects when needed
- Used for: remote workers, travelling employees, secure access

### 2.3 Client-to-Site VPN
Same as Remote Access VPN — individual client connects to a site/network.

### 2.4 VPN Concentrator
Dedicated hardware/software device that terminates many simultaneous VPN connections — used in large enterprises where a standard firewall can't handle the load.

---

## 3. VPN Protocols

### 3.1 IPSec — Internet Protocol Security
Operates at **Layer 3 (Network Layer)**. Most common for site-to-site VPNs.

**Two core protocols:**
```
AH — Authentication Header
  ├── Provides: Integrity + Authentication
  ├── Does NOT provide: Encryption (no confidentiality)
  └── Protocol number: 51

ESP — Encapsulating Security Payload
  ├── Provides: Encryption + Integrity + Authentication
  ├── This is what you actually want — use ESP
  └── Protocol number: 50
```

**Two modes:**
```
Transport Mode:
  [ IP Header | ESP Header | Payload | ESP Trailer ]
  Only the payload is encrypted — original IP header visible
  Used for: host-to-host communication

Tunnel Mode:
  [ New IP Header | ESP Header | Original IP Header | Payload | ESP Trailer ]
  Entire original packet encrypted and encapsulated
  Used for: site-to-site VPNs (hides internal IP addresses)
```

**IPSec key exchange — IKE (Internet Key Exchange):**
```
IKEv1 Phase 1 → Establish secure channel, authenticate peers
IKEv1 Phase 2 → Negotiate IPSec SA (Security Association), establish tunnel
IKEv2         → Faster, more reliable, supports MOBIKE (mobile devices)
```

**Ports/Protocols:**
```
IKE    → UDP 500
IKE NAT-T → UDP 4500 (when NAT device detected)
ESP    → IP Protocol 50
AH     → IP Protocol 51
```

### 3.2 SSL/TLS VPN
Operates at **Application Layer**. Uses standard HTTPS (port 443).

```
Types:
  ├── Clientless SSL VPN — browser-based, no client needed
  │   Access web apps through browser portal
  │
  └── Full tunnel SSL VPN — lightweight client required
      All traffic routed through VPN tunnel
      Example: Cisco AnyConnect, GlobalProtect, OpenVPN
```

**Advantages over IPSec:**
- Works through firewalls (port 443 rarely blocked)
- Easier to deploy for remote users
- No special client for clientless mode
- NAT-friendly

### 3.3 OpenVPN
Open-source VPN using SSL/TLS for key exchange and encryption.

```
Protocol:   TCP or UDP (UDP preferred for performance)
Port:       1194 (default) or 443 (for firewall bypass)
Encryption: AES-256-GCM
Auth:       Certificates, username/password, or both
Code:       ~100,000 lines (complex — larger attack surface)
```

### 3.4 WireGuard
Modern, lightweight VPN protocol — rapidly becoming the new standard.

```
Protocol:   UDP only
Port:       51820 (default)
Encryption: ChaCha20 (symmetric), Curve25519 (key exchange)
Code:       ~4,000 lines (simple — easier to audit)
Speed:      Significantly faster than OpenVPN and IPSec
Auth:       Public key cryptography only
```

**WireGuard vs OpenVPN:**
| Feature | WireGuard | OpenVPN |
|---------|-----------|---------|
| Code size | ~4,000 lines | ~100,000 lines |
| Speed | Very fast | Moderate |
| Protocol | UDP only | TCP or UDP |
| Setup | Simple | Complex |
| Audit surface | Small | Large |

### 3.5 L2TP/IPSec
Layer 2 Tunneling Protocol combined with IPSec encryption.

```
L2TP   → Creates the tunnel (no encryption itself)
IPSec  → Provides encryption for L2TP traffic
Port:    UDP 1701 (L2TP) + UDP 500/4500 (IPSec)
Status:  Being replaced by IKEv2 and WireGuard
```

### 3.6 PPTP — Point-to-Point Tunneling Protocol
**Status: BROKEN — never use**
```
Microsoft's old VPN protocol — MS-CHAPv2 authentication is broken
Crackable with tools like chapcrack
Only use: legacy systems with no alternatives (then upgrade immediately)
```

### 3.7 Protocol Comparison

| Protocol | Layer | Port | Encryption | Speed | Use case |
|----------|-------|------|-----------|-------|---------|
| IPSec/IKEv2 | 3 | UDP 500/4500 | AES | Fast | Site-to-site, mobile |
| SSL/TLS VPN | 7 | TCP 443 | AES | Moderate | Remote access |
| OpenVPN | 7 | UDP 1194/TCP 443 | AES-256 | Moderate | Remote access |
| WireGuard | 3 | UDP 51820 | ChaCha20 | Very fast | All use cases |
| L2TP/IPSec | 2+3 | UDP 1701/500 | AES | Moderate | Legacy remote access |
| PPTP | 2 | TCP 1723 | Broken | Fast | Never — broken |

---

## 4. Split Tunneling

Controls which traffic goes through the VPN tunnel and which goes directly to the internet.

### Full Tunnel
```
All traffic → VPN tunnel → Corporate network → Internet
  Pros: Full visibility for SOC, consistent security policy
  Cons: Higher bandwidth load on VPN, slower for users
```

### Split Tunnel
```
Corporate traffic  → VPN tunnel → Corporate network
Personal traffic   → Direct internet connection (no VPN)
  Pros: Less VPN load, faster browsing for users
  Cons: SOC blind spot — personal traffic not monitored
         Malware on endpoint can use non-VPN path
```

### Inverse Split Tunnel
```
Specific traffic   → Direct internet (bypass VPN)
Everything else    → VPN tunnel
  Used for: allowing streaming services to bypass VPN
```

---

## 5. VPN Authentication

| Method | Description | Security level |
|--------|-------------|---------------|
| **Username + Password** | Basic credential auth | Low — phishable |
| **MFA** | Password + OTP/push notification | High — recommended |
| **Certificate-based** | Digital certificates on device | Very high |
| **Pre-shared key (PSK)** | Shared secret between endpoints | Medium — site-to-site |
| **RADIUS integration** | Centralized auth server | High — enterprise |
| **LDAP/Active Directory** | Corporate directory auth | High — enterprise |

---

## 6. Key Terms & Terminology

| Term | Meaning |
|------|---------|
| **VPN** | Virtual Private Network — encrypted tunnel over public network |
| **Tunnel** | Encapsulated, encrypted connection between two endpoints |
| **IPSec** | IP Security — Layer 3 VPN protocol suite |
| **ESP** | Encapsulating Security Payload — IPSec encryption component |
| **AH** | Authentication Header — IPSec integrity without encryption |
| **IKE** | Internet Key Exchange — IPSec key negotiation protocol |
| **SA** | Security Association — parameters for an IPSec connection |
| **SSL VPN** | VPN using TLS — runs on port 443 |
| **WireGuard** | Modern lightweight VPN protocol |
| **Split tunneling** | Only some traffic goes through VPN |
| **VPN concentrator** | Dedicated device terminating many VPN connections |
| **Always-on VPN** | VPN connects automatically — no user action |
| **Kill switch** | Blocks internet if VPN drops — prevents traffic leak |
| **DNS leak** | DNS queries bypassing VPN — exposes browsing to ISP |
| **Full tunnel** | All traffic through VPN |
| **MOBIKE** | IKEv2 extension — maintains VPN when IP changes (mobile) |

---

## 7. 🔐 SOC Analyst Relevance

VPN logs are among the most valuable sources for SOC investigations — they reveal who connected, from where, and when.

### Key VPN Threats

| Threat | Description | Detection |
|--------|-------------|-----------|
| **Impossible travel** | VPN login from Pakistan then USA 10 min later | Geo-velocity alert in SIEM |
| **Credential stuffing** | Automated login attempts using leaked credentials | Multiple failed auth from many IPs |
| **Brute force** | Password guessing against VPN endpoint | Rapid failed logins from single IP |
| **VPN credential phishing** | Fake VPN portal steals credentials | New domain mimicking VPN URL |
| **Attacker using commercial VPN** | Hides real IP using NordVPN, ExpressVPN etc. | IP reputation check — known VPN exit node |
| **Unauthorized VPN client** | Rogue VPN bypassing corporate controls | Unexpected VPN protocols on network |
| **Split tunnel abuse** | Malware exfiltrates via non-VPN path | Endpoint detection, not network-visible |
| **Off-hours access** | Legitimate account used at 3am | Time-based anomaly alert |
| **New country login** | First time user logged in from new country | Geo-location baseline alert |

### VPN Log Fields SOC Analysts Read

```
Timestamp   | Username | Source IP | Source Country | 
Destination | Auth result | Session duration | Bytes transferred |
MFA used    | Device type | VPN protocol
```

### SIEM Queries for VPN Monitoring

```
# Impossible travel — same user from two countries within 1 hour
index=vpn event=login | sort by user, timestamp
| streamstats window=2 current=t values(src_country) as countries by user
| where mvcount(countries) > 1

# Brute force — many failed logins from single IP
index=vpn event=auth_failed
| stats count by src_ip | where count > 20

# Credential stuffing — many failed logins across many accounts
index=vpn event=auth_failed
| stats dc(username) as accounts by src_ip | where accounts > 10

# Off-hours VPN access (outside 8am-8pm)
index=vpn event=login
| where hour(timestamp) < 8 OR hour(timestamp) > 20

# First-time login from new country
index=vpn event=login
| stats earliest(timestamp) as first_seen by username, src_country
| where first_seen > relative_time(now(), "-1d")

# Long VPN sessions (possible persistence)
index=vpn event=session_end duration > 86400

# Known commercial VPN exit nodes (attacker hiding origin)
index=vpn | lookup vpn_exit_nodes src_ip OUTPUT is_vpn_exit
| where is_vpn_exit=true
```

### VPN in the Attack Kill Chain

```
Reconnaissance    →  Attacker identifies VPN endpoint (Shodan, certificate search)
Initial Access    →  Credential stuffing / brute force / phished credentials
Persistence       →  Long VPN session, off-hours access
Lateral Movement  →  VPN gives access to internal network — pivot from there
Exfiltration      →  Data leaves through VPN tunnel (hard to inspect if full tunnel)
```

> **Real SOC scenario:** SIEM fires impossible travel alert — user "m.zafar" logged into VPN from Multan at 09:00, then from Germany at 09:14. Impossible to physically travel that fast. SOC analyst immediately forces session termination, locks the account, contacts the user to confirm compromise, resets credentials, enables MFA if not already active, and investigates what was accessed during the German session.

---

## 8. Common Interview Questions

**Q1. What is a VPN and how does it work?**
> A VPN creates an encrypted tunnel between a device or network and a remote network over the internet. Data is encrypted before leaving the source, travels encrypted across the public internet, and is decrypted at the destination. This ensures confidentiality, integrity, and authentication while masking internal network details from outside observers.

**Q2. What is the difference between site-to-site and remote access VPN?**
> Site-to-site VPN connects two entire networks permanently — configured on routers/firewalls, transparent to users. Remote access VPN connects individual users to a corporate network on demand — requires a VPN client on the user's device and user authentication before the tunnel is established.

**Q3. What is the difference between IPSec Transport mode and Tunnel mode?**
> Transport mode encrypts only the packet payload — the original IP header remains visible. It's used for host-to-host encryption. Tunnel mode encrypts the entire original packet (including IP header) and wraps it in a new IP header — hiding internal addresses. Tunnel mode is used for site-to-site VPNs where internal IP addresses must be hidden.

**Q4. What is split tunneling and what are its security implications?**
> Split tunneling routes only corporate traffic through the VPN while personal traffic goes directly to the internet. The security implication is that SOC teams lose visibility into the non-VPN traffic — malware on the endpoint can communicate over the direct internet path without being seen. Full tunneling gives complete SOC visibility but increases bandwidth load.

**Q5. What is an impossible travel alert and how would you investigate it?**
> An impossible travel alert fires when the same user account logs in from two geographically distant locations within a timeframe that makes physical travel impossible. Investigation: confirm the user didn't use a VPN or proxy themselves, check what was accessed during both sessions, terminate suspicious sessions, lock the account, reset credentials, and review for lateral movement within the corporate network.

**Q6. Why is WireGuard considered better than OpenVPN?**
> WireGuard has approximately 4,000 lines of code vs OpenVPN's 100,000+ — making it far easier to audit for vulnerabilities. It's significantly faster, simpler to configure, and uses modern cryptography (ChaCha20, Curve25519). Less code means a smaller attack surface and fewer potential vulnerabilities.

---

## 9. Quick Revision Summary

```
VPN types:
  Site-to-site    →  Two networks connected permanently (IPSec tunnels)
  Remote access   →  Individual users connect to corporate network on demand

VPN protocols:
  IPSec           →  Layer 3, AH + ESP, Transport vs Tunnel mode
  SSL/TLS VPN     →  Port 443, browser-friendly, remote access
  OpenVPN         →  Open source, SSL/TLS based, ~100k lines
  WireGuard       →  Modern, fast, ~4k lines, UDP 51820
  L2TP/IPSec      →  Legacy — being replaced
  PPTP            →  BROKEN — never use

IPSec components:
  AH   →  Integrity + Auth (no encryption)
  ESP  →  Encryption + Integrity + Auth (use this)
  IKE  →  Key exchange (IKEv2 preferred)

Split tunneling:
  Full tunnel     →  All traffic through VPN (SOC visibility ✅)
  Split tunnel    →  Only corporate traffic through VPN (SOC blind spot ⚠️)

SOC focus:
  Key threats     →  Impossible travel, brute force, credential stuffing,
                      off-hours access, new country logins
  Key logs        →  Auth events, session duration, source IP, geo-location
  Best practice   →  MFA on all VPN access — non-negotiable
```

---

*Previous topic → [Wireless Networking](./notes.md)*
*Next topic → [Network Monitoring](../07-Monitoring/notes.md)*
