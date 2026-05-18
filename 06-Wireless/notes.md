# 📶 Wireless Networking — Full Deep Dive

> **Source:** Professor Messer | **Topic #:** 14 | **Module:** 3 | **Status:** ✅ Complete

---

## 1. Definition

**Wireless networking** allows devices to connect to a network without physical cables, using radio frequency (RF) signals transmitted through the air. The most common wireless standard is **Wi-Fi (IEEE 802.11)**, operating in the 2.4GHz, 5GHz, and 6GHz frequency bands.

> Wireless networks trade the security of physical containment for convenience — signals travel through walls, floors, and into public spaces, making wireless one of the most exposed attack surfaces in any environment.

---

## 2. Wi-Fi Standards — IEEE 802.11

| Standard | Name | Frequency | Max Speed | Range | Notes |
|----------|------|-----------|-----------|-------|-------|
| **802.11a** | Wi-Fi 1 | 5GHz | 54 Mbps | Short | Less interference, shorter range |
| **802.11b** | Wi-Fi 2 | 2.4GHz | 11 Mbps | Long | First widely adopted — very slow |
| **802.11g** | Wi-Fi 3 | 2.4GHz | 54 Mbps | Long | Combined speed of a + range of b |
| **802.11n** | Wi-Fi 4 | 2.4/5GHz | 600 Mbps | Long | MIMO — multiple antennas |
| **802.11ac** | Wi-Fi 5 | 5GHz | 3.5 Gbps | Medium | MU-MIMO, beamforming |
| **802.11ax** | Wi-Fi 6 | 2.4/5GHz | 9.6 Gbps | Long | OFDMA, dense environments |
| **802.11ax** | Wi-Fi 6E | 2.4/5/6GHz | 9.6 Gbps | Medium | Adds 6GHz band — less congestion |
| **802.11be** | Wi-Fi 7 | 2.4/5/6GHz | 46 Gbps | Long | Multi-Link Operation (MLO) |

### 2.4GHz vs 5GHz vs 6GHz

| Band | Range | Speed | Congestion | Wall penetration |
|------|-------|-------|------------|-----------------|
| **2.4GHz** | Long | Lower | High (shared with Bluetooth, microwaves) | Excellent |
| **5GHz** | Medium | Higher | Lower | Good |
| **6GHz** | Short | Highest | Very low (new band) | Poor |

---

## 3. Wireless Network Components

| Component | Role |
|-----------|------|
| **Access Point (AP)** | Transmits/receives Wi-Fi signals — connects wireless clients to wired network |
| **Wireless Controller (WLC)** | Centrally manages multiple APs in enterprise — config, monitoring, roaming |
| **SSID** | Service Set Identifier — the name of the wireless network |
| **BSSID** | Basic SSID — MAC address of a specific AP |
| **BSS** | Basic Service Set — single AP + connected clients |
| **ESS** | Extended Service Set — multiple APs with same SSID — seamless roaming |
| **IBSS** | Independent BSS — ad-hoc network, device-to-device, no AP |
| **Antenna** | Omnidirectional (all directions) or directional (focused beam) |
| **Channel** | Specific frequency slice within a band — avoid overlap |

### Channel Management
```
2.4GHz channels (11 in US, 13 in EU):
  Non-overlapping: Channels 1, 6, 11
  Overlapping channels cause interference — use only 1, 6, 11

5GHz: 24+ non-overlapping channels — much less congestion
6GHz: 59 non-overlapping 80MHz channels — cleanest band
```

---

## 4. Wireless Security Protocols

### 4.1 WEP — Wired Equivalent Privacy (1997)
**Status: BROKEN — never use**

```
Encryption:  RC4 stream cipher
Key length:  40-bit or 104-bit
IV length:   24-bit (too short — reused rapidly)
Auth:        Open system or shared key

Why it fails:
  ├── 24-bit IV = only 16 million values → IVs repeat frequently
  ├── Repeated IVs allow statistical analysis → key recovery
  ├── Tools like Aircrack-ng crack WEP in minutes
  └── Completely replaced — no valid use case
```

### 4.2 WPA — Wi-Fi Protected Access (2003)
**Status: WEAK — avoid**

```
Encryption:  TKIP (Temporal Key Integrity Protocol)
             RC4 with per-packet key mixing — temporary fix over WEP
Auth:        PSK (Personal) or 802.1X (Enterprise)

Why it's weak:
  ├── TKIP still uses RC4 — inherits weaknesses
  ├── TKIP deprecated in 802.11 standard (2012)
  └── Vulnerable to TKIP attacks
```

### 4.3 WPA2 — Wi-Fi Protected Access 2 (2004)
**Status: GOOD — currently widely deployed**

```
Encryption:  AES-CCMP (Counter Mode CBC-MAC Protocol)
             AES = Advanced Encryption Standard (128-bit)
Auth modes:
  ├── WPA2-Personal (PSK): Shared password for all users
  └── WPA2-Enterprise (802.1X): Individual credentials via RADIUS server

Vulnerability:
  ├── KRACK (Key Reinstallation Attack) — 2017 — patched on most devices
  ├── PSK mode: capture 4-way handshake → offline dictionary attack
  └── PMKID attack — captures single frame for offline cracking
```

### 4.4 WPA3 — Wi-Fi Protected Access 3 (2018)
**Status: BEST — current standard**

```
Encryption:  AES-GCMP-256 (stronger than WPA2)
Auth:        SAE (Simultaneous Authentication of Equals)
             Also called Dragonfly handshake

Key improvements over WPA2:
  ├── SAE replaces PSK — immune to offline dictionary attacks
  │   Even if attacker captures handshake, cannot brute force offline
  ├── Forward secrecy — unique session keys per connection
  │   Past sessions cannot be decrypted if key is later compromised
  ├── Protected Management Frames (PMF) — mandatory
  │   Prevents deauthentication attacks
  └── 192-bit security mode for enterprise (WPA3-Enterprise)

WPA3 modes:
  WPA3-Personal   →  SAE handshake, forward secrecy
  WPA3-Enterprise →  192-bit AES-GCMP, stronger auth
  Enhanced Open   →  OWE (Opportunistic Wireless Encryption) for open networks
                     Encrypts open Wi-Fi without passwords
```

### 4.5 Protocol Comparison

| Feature | WEP | WPA | WPA2 | WPA3 |
|---------|-----|-----|------|------|
| Year | 1997 | 2003 | 2004 | 2018 |
| Encryption | RC4 | TKIP/RC4 | AES-CCMP | AES-GCMP-256 |
| Handshake | Weak | 4-way | 4-way | SAE (Dragonfly) |
| Offline crack | Yes (minutes) | Yes | Yes (PSK) | No |
| Forward secrecy | No | No | No | Yes |
| Use today | Never | No | Yes (transitioning) | Yes — recommended |

---

## 5. Wireless Authentication

### 5.1 Personal Mode (PSK)
- Single pre-shared key (password) for all users
- Simple to set up — home and small office
- Security weakness: one compromised device = all traffic potentially exposed
- WPA2-PSK: vulnerable to offline handshake cracking
- WPA3-Personal: SAE prevents offline cracking

### 5.2 Enterprise Mode (802.1X + RADIUS)
- Each user has individual credentials
- Authentication flow:

```
Wireless Client → AP → RADIUS Server → Authentication
      │              │         │
  Supplicant    Authenticator  Auth Server

EAP (Extensible Authentication Protocol) carries credentials:
  ├── EAP-TLS    →  Certificate-based — most secure
  ├── PEAP       →  Password in TLS tunnel — common enterprise
  ├── EAP-TTLS   →  Tunneled TLS — flexible inner auth
  └── EAP-FAST   →  Cisco proprietary — PAC-based
```

**Why enterprise mode matters for SOC:**
- Individual user accountability — know exactly who connected when
- Compromised password = one user affected, not entire network
- Certificate-based EAP-TLS = immune to credential attacks

### 5.3 Captive Portal
Web page that users must interact with before getting network access — used for guest Wi-Fi, hotels, airports.

```
Device connects → Browser redirected to captive portal
→ User accepts terms / enters credentials
→ Firewall rule opens for that MAC/IP
→ Internet access granted
```

---

## 6. Wireless Attacks

### 6.1 Evil Twin / Rogue Access Point
```
Attacker sets up AP with same SSID as legitimate network
Higher signal strength → clients connect to attacker's AP
All traffic passes through attacker → MITM

Detection:
  ├── WIDS (Wireless IDS) detects duplicate SSIDs
  ├── Unexpected new BSSID for known SSID
  └── Clients disconnecting and reconnecting rapidly
```

### 6.2 Deauthentication Attack
```
802.11 management frames (including deauth) are unauthenticated in WPA2
Attacker sends spoofed deauth frames to disconnect clients from AP
Clients reconnect → attacker captures WPA2 4-way handshake
Handshake used for offline dictionary/brute force attack

Prevention: WPA3 — Protected Management Frames (PMF) mandatory
Detection: WIDS alerts on deauth flood, client disconnect patterns
```

### 6.3 WPA2 Handshake Capture + Offline Cracking
```
Step 1 — Attacker monitors wireless traffic (monitor mode)
Step 2 — Sends deauth frames to force client reconnection
Step 3 — Captures 4-way WPA2 handshake on reconnection
Step 4 — Takes handshake offline
Step 5 — Dictionary/brute force attack with Hashcat or Aircrack-ng
Step 6 — If password in wordlist → cracked

Tools: Aircrack-ng, Hashcat, hcxdumptool
Prevention: Strong random passphrases (20+ chars), switch to WPA3
```

### 6.4 PMKID Attack
```
Newer WPA2 attack — doesn't require capturing a client handshake
Captures PMKID from a single beacon/probe frame
Offline dictionary attack against PMKID
No need to wait for client to connect

Tool: hcxdumptool + Hashcat
Prevention: WPA3-Personal (SAE eliminates PMKID vulnerability)
```

### 6.5 WPS Attacks
```
WPS (Wi-Fi Protected Setup) — 8-digit PIN for easy device pairing
PIN can be brute forced — only 11,000 combinations (due to design flaw)
Pixie Dust attack — offline WPS PIN cracking in seconds on some APs

Prevention: Disable WPS entirely on all APs
```

### 6.6 Wardriving
```
Driving/walking around while scanning for open or poorly secured Wi-Fi networks
Used for reconnaissance — mapping network targets
Tools: Kismet, NetStumbler, WiFi Analyzer
Legal status: Varies by jurisdiction — passive scanning vs active connection
```

---

## 7. Wireless Security Best Practices

```
✅ Use WPA3-Personal or WPA3-Enterprise
✅ Strong random passphrase (20+ characters, not dictionary words)
✅ Use WPA2-Enterprise (802.1X) for corporate networks
✅ Disable WPS on all access points
✅ Separate guest Wi-Fi on isolated VLAN
✅ Implement WIDS (Wireless Intrusion Detection System)
✅ Disable SSID broadcast for sensitive networks (security through obscurity — minimal benefit)
✅ Use certificate-based EAP-TLS for enterprise authentication
✅ Enable Protected Management Frames (PMF) on WPA2 networks
✅ Conduct regular wireless site surveys and rogue AP scans
✅ Segment IoT devices on dedicated wireless VLAN
✅ Monitor for unexpected BSSIDs on known SSIDs
```

---

## 8. Key Terms & Terminology

| Term | Meaning |
|------|---------|
| **SSID** | Network name broadcasted by AP |
| **BSSID** | MAC address of specific AP |
| **AP** | Access Point — wireless radio transmitter/receiver |
| **WLC** | Wireless LAN Controller — centrally manages APs |
| **WEP** | Broken wireless security protocol — never use |
| **WPA2** | Current widely-deployed wireless security (AES-CCMP) |
| **WPA3** | Latest wireless security standard (SAE, forward secrecy) |
| **PSK** | Pre-Shared Key — single password for all users |
| **SAE** | Simultaneous Authentication of Equals — WPA3 handshake |
| **802.1X** | Port-based authentication framework — enterprise Wi-Fi |
| **RADIUS** | Remote Authentication Dial-In User Service — enterprise auth server |
| **EAP** | Extensible Authentication Protocol — carries auth credentials |
| **PMF** | Protected Management Frames — prevents deauth attacks |
| **Evil twin** | Rogue AP mimicking legitimate AP SSID |
| **Deauth attack** | Forcing clients to disconnect using spoofed management frames |
| **WIDS** | Wireless Intrusion Detection System |
| **WPS** | Wi-Fi Protected Setup — vulnerable PIN-based setup |
| **TKIP** | Temporal Key Integrity Protocol — WPA encryption (weak) |
| **AES-CCMP** | Strong AES encryption used in WPA2 |
| **Forward secrecy** | Past sessions unreadable even if key is later compromised |

---

## 9. 🔐 SOC Analyst Relevance

### Key Wireless Threats SOC Monitors

| Threat | Indicators | SOC Action |
|--------|-----------|-----------|
| **Evil twin AP** | Duplicate SSID with different BSSID, signal anomaly | WIDS alert → locate rogue AP → disable |
| **Deauth flood** | Mass client disconnects, deauth frame spike | WIDS alert → identify attacker MAC → block |
| **Rogue AP** | Unknown BSSID in airspace | Physical location scan → remove |
| **WPA2 handshake capture** | Deauth followed by client reconnect + handshake | Investigate source, check for cracking tools |
| **Unauthorized client** | Unknown MAC on corporate SSID | 802.1X rejects → alert → investigate device |
| **WPS brute force** | Repeated WPS PIN attempts | Disable WPS → block source |
| **Weak encryption** | WEP or WPA detected on network | Immediate remediation required |

### SIEM Queries for Wireless Security
```
# Detect rogue AP — unknown BSSID on known SSID
index=wids event=new_ap ssid="CorpWiFi" NOT bssid IN (authorized_bssid_list)

# Deauthentication flood attack
index=wids event=deauth_frame | stats count by src_mac | where count > 50

# Client connecting to unauthorized SSID
index=wireless ssid NOT IN (authorized_ssid_list)

# WPS brute force attempts
index=wireless event=wps_attempt | stats count by src_mac | where count > 5

# Detect clients using WEP (should be zero)
index=wireless encryption_type=WEP
```

---

## 10. Common Interview Questions

**Q1. What is the difference between WPA2-Personal and WPA2-Enterprise?**
> WPA2-Personal uses a pre-shared key (password) shared by all users — simpler but less secure. WPA2-Enterprise uses 802.1X with a RADIUS server, giving each user individual credentials. Enterprise mode provides better accountability, allows per-user access revocation, and is far more secure for corporate environments.

**Q2. Why is WEP considered completely broken?**
> WEP uses a 24-bit initialization vector (IV) that repeats after ~5,000 packets on a busy network. Statistical analysis of repeated IVs allows recovery of the RC4 encryption key. Tools like Aircrack-ng crack WEP in minutes regardless of password strength. It has no valid use case today.

**Q3. What is an evil twin attack and how is it detected?**
> An evil twin is a rogue access point broadcasting the same SSID as a legitimate AP — often with higher signal strength to attract connections. All traffic passes through the attacker enabling MITM. Detected by WIDS monitoring for duplicate SSIDs with unexpected BSSIDs, or anomalous signal behavior patterns.

**Q4. What makes WPA3 better than WPA2?**
> WPA3 introduces SAE (Simultaneous Authentication of Equals) which replaces the 4-way PSK handshake — making offline dictionary attacks impossible even if someone captures the handshake. It also provides forward secrecy (unique session keys mean past sessions can't be decrypted), mandatory Protected Management Frames (preventing deauth attacks), and stronger 256-bit encryption.

**Q5. What is a deauthentication attack and how does WPA3 prevent it?**
> A deauth attack sends spoofed 802.11 deauthentication management frames to disconnect clients from their AP — in WPA2, these frames are unauthenticated. WPA3 mandates Protected Management Frames (PMF), which cryptographically authenticate management frames, making spoofed deauth frames invalid.

---

## 11. Quick Revision Summary

```
Wi-Fi standards:
  802.11n  (Wi-Fi 4)  →  2.4/5GHz, 600Mbps, MIMO
  802.11ac (Wi-Fi 5)  →  5GHz, 3.5Gbps, MU-MIMO
  802.11ax (Wi-Fi 6)  →  2.4/5/6GHz, 9.6Gbps, OFDMA
  802.11ax (Wi-Fi 6E) →  Adds 6GHz band

Security protocols:
  WEP   →  Broken (RC4, weak IV) — never use
  WPA   →  Weak (TKIP) — avoid
  WPA2  →  Good (AES-CCMP) — still widely used
  WPA3  →  Best (SAE, forward secrecy, PMF) — recommended

Auth modes:
  Personal/PSK     →  Shared password — home/small office
  Enterprise/802.1X →  Individual credentials via RADIUS — corporate

Key attacks:
  Evil twin        →  Rogue AP with same SSID
  Deauth attack    →  Force disconnect → capture handshake
  WPA2 cracking    →  Offline dictionary on captured handshake
  PMKID attack     →  Crack without capturing full handshake
  WPS attack       →  Brute force 8-digit PIN (11,000 combos)

SOC focus:
  Monitor          →  WIDS alerts, rogue AP scans, deauth floods
  Best defense     →  WPA3, 802.1X enterprise, disable WPS, PMF enabled
  Key log source   →  Wireless controller logs, WIDS alerts
```

---

*Previous topic → [Network Protocols](../04-Protocols/network-protocols.md)*
*Next topic → [VPNs](./vpns.md)*
