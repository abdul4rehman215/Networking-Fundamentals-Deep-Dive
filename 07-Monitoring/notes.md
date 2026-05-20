# 📡 Network Monitoring — Full Deep Dive

> **Source:** Professor Messer | **Topic #:** 16 | **Module:** 3 | **Status:** ✅ Complete

---

## 1. Definition

**Network monitoring** is the continuous, systematic process of observing network traffic, device health, performance metrics, and security events to detect anomalies, outages, and threats in real time.

> You cannot defend what you cannot see. Network monitoring is the foundation of every SOC — it turns raw network activity into actionable intelligence.

**Core goals:**
- **Visibility** — know what's happening on the network at all times
- **Availability** — detect and respond to outages before users notice
- **Security** — identify threats, anomalies, and policy violations
- **Performance** — ensure network operates within expected parameters
- **Compliance** — maintain audit trails for regulatory requirements

---

## 2. Types of Network Monitoring

| Type | What it monitors | Primary use |
|------|-----------------|-------------|
| **Packet capture** | Full network traffic content | Deep investigation, forensics |
| **Flow monitoring** | Traffic metadata (who, what, when, how much) | Anomaly detection, trending |
| **Device monitoring** | Hardware health, CPU, memory, interfaces | Availability, performance |
| **Log monitoring** | Events from network devices and systems | Security, audit, compliance |
| **Bandwidth monitoring** | Traffic volume over time | Capacity planning, DDoS detection |
| **Availability monitoring** | Is a device/service up or down? | NOC, uptime tracking |

---

## 3. Packet Capture & Analysis

### 3.1 Wireshark
The most widely used packet analyzer — open source, GUI-based, cross-platform.

```
What Wireshark captures:
  ├── Every packet crossing the interface
  ├── Full packet content — headers and payload
  ├── Decoded protocol details at every OSI layer
  ├── Timestamps with microsecond precision
  └── Source/destination MAC, IP, port for every packet
```

**Key Wireshark features:**

| Feature | Description |
|---------|-------------|
| **Display filters** | Filter captured traffic to show only what's relevant |
| **Capture filters** | Only capture specific traffic (saves storage) |
| **Protocol dissectors** | Automatically decodes 1000+ protocols |
| **Follow stream** | Reconstruct full TCP/UDP conversation |
| **Statistics** | Protocol hierarchy, conversations, endpoints |
| **Export objects** | Extract files transferred over HTTP, SMB, FTP |
| **Coloring rules** | Color-code packets by type for quick identification |

**Essential Wireshark display filters for SOC:**
```
# Filter by IP address
ip.addr == 192.168.1.100
ip.src == 10.0.0.5
ip.dst == 8.8.8.8

# Filter by protocol
tcp
udp
dns
http
icmp
arp

# Filter by port
tcp.port == 443
tcp.dstport == 22
udp.port == 53

# Filter TCP flags
tcp.flags.syn == 1 && tcp.flags.ack == 0    ← SYN scan detection
tcp.flags.reset == 1                         ← Connection resets
tcp.flags.fin == 1                           ← Connection teardowns

# Filter HTTP
http.request.method == "POST"
http.response.code == 200
http.host contains "suspicious"

# Filter DNS
dns.qry.name contains "evil"
dns.flags.rcode == 3                         ← NXDOMAIN responses

# Filter large packets (possible exfiltration)
frame.len > 1400

# Filter ARP (detect ARP poisoning)
arp.duplicate-address-detected

# Combine filters
ip.src == 192.168.1.50 && tcp.dstport == 443 && tcp.flags.syn == 1
```

### 3.2 tcpdump
Command-line packet capture — used on Linux servers and network devices where GUI isn't available.

```bash
# Capture all traffic on interface eth0
tcpdump -i eth0

# Capture traffic to/from specific IP
tcpdump -i eth0 host 192.168.1.100

# Capture traffic on specific port
tcpdump -i eth0 port 443

# Save capture to file (for Wireshark analysis)
tcpdump -i eth0 -w capture.pcap

# Read from file
tcpdump -r capture.pcap

# Capture DNS traffic with verbose output
tcpdump -i eth0 -v port 53

# Capture and show packet contents
tcpdump -i eth0 -X port 80

# Capture SYN packets only (port scan detection)
tcpdump -i eth0 'tcp[tcpflags] & tcp-syn != 0'
```

### 3.3 Port Mirroring / SPAN
To capture traffic, the monitoring tool must see it. Two methods:

```
Port Mirroring (SPAN — Switched Port Analyzer):
  Switch configured to copy all traffic from monitored ports
  to a dedicated monitoring port where Wireshark/IDS is connected.

Network TAP (Test Access Point):
  Hardware device physically inserted into a cable
  Passively copies all traffic — zero impact on network
  Cannot be detected or disabled remotely
  Preferred for critical monitoring points
```

---

## 4. Flow Monitoring

### 4.1 NetFlow
Developed by Cisco — records **metadata** about network flows rather than full packet content.

```
A "flow" = all packets sharing same:
  ├── Source IP
  ├── Destination IP
  ├── Source port
  ├── Destination port
  ├── Protocol
  └── (optionally) TOS/DSCP value

NetFlow record captures:
  ├── Start/end timestamp
  ├── Source IP + port
  ├── Destination IP + port
  ├── Protocol
  ├── Bytes transferred
  ├── Packets transferred
  └── TCP flags seen
```

**NetFlow vs Full Packet Capture:**
| Feature | NetFlow | Full PCAP |
|---------|---------|-----------|
| Storage required | Very low (metadata only) | Very high (full content) |
| Privacy | No payload content | Full content captured |
| Speed | Real-time at scale | Resource intensive |
| Encryption blind spot | Not affected | Can't see inside TLS |
| Forensic detail | Limited | Complete |
| Best for | Trending, anomaly detection | Incident investigation |

### 4.2 NetFlow Variants

| Protocol | Developer | Notes |
|----------|-----------|-------|
| **NetFlow v5** | Cisco | Most widely supported, IPv4 only |
| **NetFlow v9** | Cisco | Template-based, IPv6 support |
| **IPFIX** | IETF standard | Based on NetFlow v9 — vendor neutral |
| **sFlow** | HP/InMon | Packet sampling — scales better |
| **J-Flow** | Juniper | Juniper's NetFlow equivalent |

### 4.3 NetFlow Analysis Tools
- **ntopng** — open source flow analysis dashboard
- **Elastic Stack** — ingest and visualize flow data
- **SolarWinds NTA** — enterprise NetFlow analysis
- **ManageEngine NetFlow Analyzer** — SMB focused
- **Scrutinizer** — Plixer's NetFlow analyzer

---

## 5. SNMP — Simple Network Management Protocol

### 5.1 Definition
**SNMP** monitors and manages network devices — routers, switches, firewalls, servers, printers — by polling them for status information and receiving alerts (traps) when thresholds are breached.

**Ports:** UDP 161 (queries), UDP 162 (traps)

### 5.2 SNMP Components

```
SNMP Manager (NMS)      →  Network Management Station — polls devices
SNMP Agent              →  Software running on monitored device — responds to queries
MIB                     →  Management Information Base — database of objects that can be monitored
OID                     →  Object Identifier — unique ID for each monitored metric
Trap                    →  Unsolicited alert sent from device to manager when event occurs
```

### 5.3 SNMP Versions

| Version | Security | Notes |
|---------|----------|-------|
| **SNMPv1** | Community string (plaintext) | Obsolete — never use |
| **SNMPv2c** | Community string (plaintext) | Still common but insecure |
| **SNMPv3** | Authentication + Encryption | Current standard — use this |

```
SNMPv3 security levels:
  noAuthNoPriv  →  No authentication, no encryption (avoid)
  authNoPriv    →  Authentication only (MD5/SHA)
  authPriv      →  Authentication + Encryption (AES) — use this
```

### 5.4 What SNMP Monitors

```
Network devices:
  ├── Interface status (up/down)
  ├── Bandwidth utilization (in/out bytes per second)
  ├── CPU and memory utilization
  ├── Error counters (CRC errors, drops, collisions)
  ├── Temperature and hardware health
  └── Routing table changes

SNMP Traps alert on:
  ├── Interface going down
  ├── CPU threshold exceeded (>90%)
  ├── Bandwidth saturation
  ├── Authentication failure
  └── Device restart/reboot
```

### 5.5 Popular SNMP Monitoring Tools
- **Nagios** — open source, highly extensible
- **Zabbix** — open source, enterprise-grade
- **PRTG** — commercial, easy GUI
- **SolarWinds NPM** — enterprise network monitoring
- **Cacti** — open source, RRDtool-based graphing
- **LibreNMS** — open source, auto-discovery

---

## 6. Syslog — Centralized Log Management

### 6.1 Definition
**Syslog** is a standard protocol for sending log messages from network devices and systems to a centralized log server. It provides a single point to collect, store, and analyze logs from the entire environment.

**Port:** UDP 514 (traditional) / TCP 514 or 6514/TLS (secure)

### 6.2 Syslog Message Format

```
<Priority> Version Timestamp Hostname AppName ProcID MsgID Message

Example:
<34>1 2026-05-20T10:14:22Z fw01 %ASA-3-106014 - Deny inbound icmp
                                                src outside:45.33.32.156
                                                dst inside:10.0.1.50

Priority = Facility × 8 + Severity
```

### 6.3 Syslog Severity Levels

| Level | Name | Description | Example |
|-------|------|-------------|---------|
| 0 | **Emergency** | System unusable | Kernel panic |
| 1 | **Alert** | Immediate action required | Loss of primary ISP |
| 2 | **Critical** | Critical conditions | Hard disk failure |
| 3 | **Error** | Error conditions | Authentication failure |
| 4 | **Warning** | Warning conditions | High CPU usage |
| 5 | **Notice** | Normal but significant | Config change |
| 6 | **Informational** | Informational messages | User login |
| 7 | **Debug** | Debug-level messages | Verbose app output |

### 6.4 Syslog Facilities

| Code | Facility |
|------|---------|
| 0 | Kernel messages |
| 4 | Security/auth messages |
| 16–23 | Local use (local0–local7) — commonly used for network devices |

### 6.5 Syslog Infrastructure

```
Network Devices (routers, switches, firewalls)
Servers (Linux, Windows)
Security tools (IDS, WAF, VPN)
        │
        ▼ (UDP/TCP 514)
Syslog Server / SIEM
  ├── rsyslog (Linux)
  ├── syslog-ng
  ├── Splunk (ingests syslog)
  └── Elastic Stack (Logstash ingests syslog)
```

---

## 7. Bandwidth Monitoring

### 7.1 MRTG — Multi Router Traffic Grapher
Classic tool that uses SNMP to poll bandwidth utilization and creates graphs.

### 7.2 Key Bandwidth Metrics

```
Throughput   →  Actual data transfer rate (Mbps)
Utilization  →  % of available bandwidth in use
Latency      →  Round-trip time (ms) — ping
Jitter       →  Variation in latency — critical for VoIP
Packet loss  →  % of packets dropped — sign of congestion or failure
```

### 7.3 Baseline
```
Baseline = normal expected values for your environment

Without baseline:
  "Traffic is at 800Mbps" — is that normal or an attack?

With baseline:
  "Normal is 200-400Mbps on weekdays. 800Mbps at 2am = anomaly = investigate"
```

---

## 8. Network Monitoring Architecture

```
                    ┌─────────────────────────────────┐
                    │         SOC Dashboard            │
                    │   Alerts │ Graphs │ Reports      │
                    └──────────────────────────────────┘
                              │
                    ┌─────────────────────────────────┐
                    │     SIEM / Log Management        │
                    │  (Splunk, Elastic, Sentinel)     │
                    └──────────────────────────────────┘
                    │              │              │
            ┌───────┘      ┌───────┘      ┌──────┘
            ▼              ▼              ▼
       Syslog Server   NetFlow        SNMP Manager
            │           Collector        │
            │              │             │
     ┌──────┴──────────────┴─────────────┴──────┐
     │     Network Infrastructure                │
     │  Routers │ Switches │ Firewalls │ Servers │
     └───────────────────────────────────────────┘
            │
     ┌──────┘
     ▼
  TAP/SPAN Port
     │
  Wireshark / IDS / NDR
```

---

## 9. Key Monitoring Tools Summary

| Tool | Type | Open Source | Best for |
|------|------|-------------|---------|
| **Wireshark** | Packet analysis | Yes | Packet-level investigation |
| **tcpdump** | Packet capture (CLI) | Yes | Server-side capture |
| **Zeek (Bro)** | Network analysis | Yes | Behavioral analysis, log generation |
| **Suricata** | IDS/IPS + NSM | Yes | Threat detection + flow logs |
| **ntopng** | Flow analysis | Yes (community) | NetFlow visualization |
| **Nagios** | Device monitoring | Yes | Availability + SNMP alerting |
| **Zabbix** | Device + app monitoring | Yes | Enterprise monitoring |
| **PRTG** | All-in-one monitoring | No | Easy setup, SMB |
| **SolarWinds NPM** | Network monitoring | No | Large enterprise |
| **Elastic Stack** | Log management | Yes | Log analysis + SIEM |
| **Graylog** | Log management | Yes | Centralized logging |

---

## 10. Key Terms & Terminology

| Term | Meaning |
|------|---------|
| **Packet capture (PCAP)** | Full recording of network traffic |
| **NetFlow** | Traffic metadata protocol — flow records |
| **IPFIX** | Vendor-neutral NetFlow standard |
| **sFlow** | Sampled flow monitoring — scales to high speeds |
| **SNMP** | Protocol for monitoring network device health |
| **MIB** | Management Information Base — SNMP object database |
| **OID** | Object Identifier — specific SNMP metric |
| **Trap** | Unsolicited SNMP alert from device to manager |
| **Syslog** | Centralized log collection protocol |
| **SPAN port** | Switch port that mirrors traffic to monitoring tool |
| **Network TAP** | Hardware that passively copies all traffic |
| **Baseline** | Normal expected values — anomaly detection reference |
| **Latency** | Round-trip time in milliseconds |
| **Jitter** | Variation in latency |
| **Throughput** | Actual data transfer rate |
| **NMS** | Network Management System — SNMP manager |
| **NDR** | Network Detection and Response — modern threat detection |
| **Promiscuous mode** | NIC mode that captures all traffic, not just addressed to it |

---

## 11. 🔐 SOC Analyst Relevance

Network monitoring IS the SOC's core function. Here's how each tool maps to SOC work:

### Wireshark in SOC Investigations
```
Use case: Alert fires for suspicious host. Pull PCAP and investigate.

SOC workflow:
  1. Filter by suspicious IP: ip.addr == 10.0.3.45
  2. Follow TCP stream to a suspicious connection
  3. Check DNS queries: dns.qry.name
  4. Look for data exfiltration: large POST requests
  5. Identify C2 beaconing: regular interval connections
  6. Export transferred files for malware analysis
```

### NetFlow in SOC Investigations
```
Use case: Detect lateral movement without full PCAP.

SOC workflow:
  1. Query flows from compromised host: src_ip=10.0.3.45
  2. Identify all internal IPs contacted
  3. Flag unusual ports or protocols
  4. Calculate data volumes — exfiltration indicator
  5. Timeline reconstruction — when did compromise start spreading?
```

### Syslog in SOC Investigations
```
Use case: Reconstruct attack timeline from device logs.

SOC workflow:
  1. Query firewall syslogs for blocked connections
  2. Cross-reference with router logs for routing changes
  3. Check switch logs for port security violations
  4. Correlate VPN logs with internal activity
  5. Build complete timeline of attacker activity
```

### SNMP in SOC Investigations
```
Use case: DDoS impact assessment.

SOC workflow:
  1. Check SNMP bandwidth graphs — identify saturation point
  2. Identify which interfaces are affected
  3. Correlate with NetFlow — identify attack traffic source
  4. Check device CPU — is it overwhelmed?
  5. Trigger mitigation — null route, ACL block
```

### SIEM Queries for Network Monitoring

```
# Detect port scan using NetFlow
index=netflow | stats dc(dst_port) as ports by src_ip | where ports > 20

# Bandwidth anomaly — traffic spike
index=snmp metric=interface_bytes_out
| timechart span=5m sum(value) | anomalydetection

# Syslog — firewall deny spike (possible scan or attack)
index=syslog facility=firewall action=deny
| timechart span=1m count | where count > 1000

# Find new connections (first time src→dst seen)
index=netflow | stats earliest(timestamp) as first_seen by src_ip, dst_ip
| where first_seen > relative_time(now(), "-1h")

# Detect large data transfers (exfiltration)
index=netflow | stats sum(bytes) as total_bytes by src_ip, dst_ip
| where total_bytes > 104857600 ← 100MB threshold
```

> **Real SOC scenario:** SNMP alert fires — core switch showing 98% bandwidth utilization. SOC analyst pulls NetFlow data → sees thousands of flows from one external IP to internal web server → identifies UDP flood DDoS. Correlates with firewall syslog → confirms attack traffic. Implements ACL block at upstream router → monitors SNMP bandwidth graph → confirms traffic drops to normal within 2 minutes.

---

## 12. Common Interview Questions

**Q1. What is the difference between packet capture and NetFlow monitoring?**
> Packet capture records the complete content of every packet — headers and payload — giving full forensic detail but requiring significant storage. NetFlow records only metadata about traffic flows (who talked to who, how much data, which ports) — far less storage but no payload content. SOC teams use NetFlow for continuous anomaly detection and packet capture for deep investigation of specific incidents.

**Q2. What is a network baseline and why is it important?**
> A baseline is the documented normal behavior of a network — typical traffic volumes, protocols in use, peak hours, standard connections. Without a baseline, anomalies are invisible — you can't identify what's abnormal if you don't know what's normal. Baselines enable SOC teams to detect DDoS attacks, data exfiltration, and lateral movement by comparing current behavior against the established norm.

**Q3. What is SNMP and what are the security differences between versions?**
> SNMP monitors network device health — CPU, bandwidth, interface status. SNMPv1 and v2c use community strings (essentially passwords) transmitted in plaintext — trivially sniffable. SNMPv3 adds proper authentication (MD5/SHA) and encryption (AES) — it's the only version that should be used in production environments.

**Q4. What is syslog and how is it used in a SOC?**
> Syslog is a standard protocol for sending log messages from network devices, servers, and security tools to a centralized server. In a SOC, syslog feeds into the SIEM — giving analysts one place to search across logs from every device in the environment. It enables timeline reconstruction during incidents, cross-device correlation, and long-term audit trails.

**Q5. What is a SPAN port and when would a SOC analyst use it?**
> A SPAN (Switched Port Analyzer) port is a switch port configured to receive a copy of all traffic passing through monitored ports. SOC analysts use it to connect packet capture tools (Wireshark, Suricata, Zeek) to the network without disrupting traffic flow. It's essential for passive network monitoring and IDS deployment.

**Q6. How would you use Wireshark to investigate a suspected C2 connection?**
> Filter by the suspicious host IP, then look for regular-interval outbound connections (beaconing pattern). Follow the TCP/UDP streams to examine the content. Check DNS queries for random or encoded domain names. Look for unusual user agents in HTTP traffic. Examine data volumes — small outbound, larger inbound is typical C2 check-in behavior. Export any transferred files for malware analysis.

---

## 13. Quick Revision Summary

```
Network monitoring tools:
  Wireshark   →  Full packet capture and analysis — GUI
  tcpdump     →  CLI packet capture — Linux/server
  NetFlow     →  Traffic metadata — who/what/when/how much
  SNMP        →  Device health monitoring (CPU, bandwidth, interfaces)
  Syslog      →  Centralized log collection from all devices
  Zeek        →  Behavioral network analysis — rich log generation

SNMP versions:
  v1/v2c  →  Plaintext community strings — insecure
  v3      →  Auth + encryption — use this

Syslog severity (0=worst, 7=debug):
  0 Emergency → 1 Alert → 2 Critical → 3 Error
  4 Warning → 5 Notice → 6 Info → 7 Debug

NetFlow vs PCAP:
  NetFlow  →  Metadata only, low storage, always-on
  PCAP     →  Full content, high storage, investigation

Key ports:
  SNMP queries  →  UDP 161
  SNMP traps    →  UDP 162
  Syslog        →  UDP 514 (TCP 514/6514 for secure)

SOC focus:
  Baseline      →  Know normal → detect abnormal
  NetFlow       →  Detect lateral movement, exfiltration, DDoS
  Wireshark     →  Deep-dive investigation of specific incidents
  Syslog/SIEM   →  Timeline reconstruction, cross-device correlation
```

---

*Previous topic → [VPNs](../06-Wireless/vpns.md)*
*Next topic → [SIEM Basics](./siem.md)*
