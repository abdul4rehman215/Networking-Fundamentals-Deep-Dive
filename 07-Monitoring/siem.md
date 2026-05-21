# 🖥️ SIEM Basics — Full Deep Dive

> **Source:** Professor Messer | **Topic #:** 17 | **Module:** 3 | **Status:** ✅ Complete

---

## 1. Definition

**SIEM (Security Information and Event Management)** is a platform that collects, normalises, correlates, and analyses log data and security events from across an entire IT environment — generating alerts when threats are detected and providing a centralised interface for investigation, threat hunting, and compliance reporting.

SIEM = **SIM** (Security Information Management) + **SEM** (Security Event Management)

| Component | Function |
|-----------|---------|
| **SIM** | Long-term log storage, indexing, search, compliance reporting |
| **SEM** | Real-time event correlation, alerting, threat detection |

> Think of a SIEM as the command centre of the SOC — all intelligence flows in, processed threats flow out as alerts, and analysts use it as their primary investigation tool.

---

## 2. How SIEM Works — Step by Step

```
Step 1 — LOG COLLECTION
  Every source sends logs to SIEM:
  ├── Firewalls (allow/deny events)
  ├── IDS/IPS (alerts and signatures)
  ├── Endpoints (Windows Event Logs, EDR)
  ├── Servers (auth logs, application logs)
  ├── Network devices (router, switch syslogs)
  ├── DNS servers (query logs)
  ├── Proxy/web (HTTP access logs)
  ├── VPN (auth and session logs)
  ├── Cloud platforms (AWS CloudTrail, Azure AD)
  └── Active Directory (login, group changes)

Step 2 — NORMALISATION
  Raw logs arrive in different formats (syslog, JSON, XML, CEF, LEEF)
  SIEM normalises all into a common schema:
  ├── Timestamp
  ├── Source IP
  ├── Destination IP
  ├── Username
  ├── Action (allow/deny/login/logout)
  ├── Event type
  └── Severity

Step 3 — INDEXING & STORAGE
  Normalised events stored in searchable index
  Enables fast queries across billions of events
  Retention policies (30 days hot, 1 year cold, 7 years archive)

Step 4 — CORRELATION
  Rules engine applies detection logic across multiple events:
  "IF 10 failed logins in 60 seconds from same IP → brute force alert"
  "IF login success AFTER 20 failures → credential stuffing success"
  "IF new process spawned by Word.exe → possible macro malware"

Step 5 — ALERTING
  Correlation rule fires → alert created
  Alert assigned severity (Critical / High / Medium / Low / Info)
  Alert routed to SOC analyst queue

Step 6 — INVESTIGATION
  SOC analyst receives alert
  Uses SIEM search to gather context
  Correlates across data sources
  Determines: True positive or false positive?

Step 7 — RESPONSE
  True positive → escalate, contain, remediate
  False positive → document, tune rule to reduce recurrence
```

---

## 3. SIEM Data Sources

### 3.1 Log Source Categories

| Category | Examples | Key events |
|----------|---------|-----------|
| **Network** | Firewall, router, switch, proxy | Allow/deny, connections, traffic volume |
| **Identity** | Active Directory, LDAP, RADIUS | Logins, lockouts, group changes, new accounts |
| **Endpoint** | Windows Event Logs, EDR (CrowdStrike, Defender) | Process creation, file changes, registry |
| **Application** | Web servers, databases, email | HTTP requests, SQL queries, mail events |
| **Cloud** | AWS CloudTrail, Azure Monitor, GCP | API calls, resource creation, config changes |
| **Security tools** | IDS/IPS, WAF, DLP, vulnerability scanner | Alerts, signatures, scan results |
| **Threat intel** | MISP, VirusTotal, commercial feeds | IOCs — IPs, domains, hashes |

### 3.2 Critical Windows Event IDs for SOC

| Event ID | Description | SOC relevance |
|----------|-------------|--------------|
| **4624** | Successful logon | Baseline, detect unusual logins |
| **4625** | Failed logon | Brute force detection |
| **4648** | Logon with explicit credentials | Pass-the-hash, lateral movement |
| **4672** | Special privileges assigned | Privilege escalation |
| **4698** | Scheduled task created | Persistence mechanism |
| **4720** | User account created | Unauthorized account creation |
| **4726** | User account deleted | Cover tracks |
| **4732** | Member added to security group | Privilege escalation |
| **4768** | Kerberos TGT requested | Kerberoasting (watch for unusual accounts) |
| **4769** | Kerberos service ticket requested | Kerberoasting |
| **4771** | Kerberos pre-auth failed | AS-REP Roasting |
| **7045** | New service installed | Persistence, malware installation |
| **1102** | Audit log cleared | Attacker covering tracks |

---

## 4. SIEM Detection Methods

### 4.1 Rule-Based Detection (Signature)
Predefined conditions that trigger when matched:

```
Example rules:
  IF failed_logins > 10 IN 60 seconds FROM same src_ip
  → Alert: Brute Force Attempt

  IF login_success AFTER failed_logins > 5 FROM same src_ip
  → Alert: Possible Successful Brute Force

  IF process_name = "cmd.exe" AND parent_process = "winword.exe"
  → Alert: Macro Execution — Possible Malware

  IF dst_port = 4444 OR dst_port = 1337 OR dst_port = 31337
  → Alert: Known C2 Port Usage

  IF event_id = 1102 (log cleared)
  → Alert: CRITICAL — Audit Log Cleared
```

### 4.2 Anomaly-Based Detection (Behavioural)
Establishes baseline, alerts on deviations:

```
Baseline:
  User "m.zafar" logs in Mon-Fri, 8am-6pm, from Multan, Pakistan

Anomaly detected:
  User "m.zafar" logs in Saturday 3am from Germany
  → Alert: Anomalous Login — Impossible Travel

Baseline:
  Server sends ~500MB/day outbound

Anomaly detected:
  Server sends 50GB in one hour
  → Alert: Data Exfiltration Suspected
```

### 4.3 Threat Intelligence Correlation
Matches observed IOCs against known malicious indicators:

```
Threat intel feeds provide:
  ├── Malicious IP addresses
  ├── Known C2 domains
  ├── Malware file hashes (MD5/SHA256)
  ├── Phishing URLs
  └── Attacker TTPs (MITRE ATT&CK)

SIEM auto-matches:
  IF src_ip or dst_ip IN (threat_intel_ip_list)
  → Alert: Connection to Known Malicious IP
```

### 4.4 UEBA — User and Entity Behaviour Analytics
Advanced ML-based detection that profiles each user/entity and scores deviations:

```
Risk score components:
  ├── Login time anomaly (+20 risk)
  ├── New geographic location (+30 risk)
  ├── Unusual data access volume (+25 risk)
  ├── Privilege escalation attempt (+40 risk)
  └── New process execution (+15 risk)

Total risk score > threshold → Alert
```

---

## 5. SIEM Query Languages

### 5.1 Splunk SPL (Search Processing Language)
```splunk
# Find all failed logins
index=windows EventCode=4625
| stats count by src_ip, user
| where count > 10
| sort -count

# Detect brute force followed by success
index=windows EventCode=4625
| stats count as failures by src_ip, user
| where failures > 5
| join user [search index=windows EventCode=4624]
| table user, src_ip, failures

# Find processes spawned by Office apps
index=endpoint parent_process IN ("winword.exe","excel.exe","powerpnt.exe")
| table timestamp, hostname, user, parent_process, process_name, command_line

# DNS tunneling — long domain names
index=dns
| eval domain_len=len(query)
| where domain_len > 50
| stats count by query, src_ip
| sort -count

# Data exfiltration — large outbound transfers
index=proxy
| stats sum(bytes_out) as total_out by src_ip, dst_ip
| where total_out > 104857600
| sort -total_out
```

### 5.2 Microsoft Sentinel KQL (Kusto Query Language)
```kql
// Failed logins - brute force detection
SecurityEvent
| where EventID == 4625
| summarize FailCount = count() by IpAddress, Account, bin(TimeGenerated, 5m)
| where FailCount > 10
| order by FailCount desc

// Successful login after failures
let failures = SecurityEvent
| where EventID == 4625
| summarize FailCount = count() by Account, IpAddress;
SecurityEvent
| where EventID == 4624
| join kind=inner failures on Account
| where FailCount > 5
| project TimeGenerated, Account, IpAddress, FailCount

// New admin account created
SecurityEvent
| where EventID == 4720
| project TimeGenerated, Account, SubjectUserName, Computer

// Impossible travel
SigninLogs
| where ResultType == 0
| project TimeGenerated, UserPrincipalName, Location, IPAddress
| sort by UserPrincipalName, TimeGenerated asc
| extend PrevLocation = prev(Location), PrevTime = prev(TimeGenerated)
| where Location != PrevLocation
| extend TimeDiff = datetime_diff('minute', TimeGenerated, PrevTime)
| where TimeDiff < 60
```

### 5.3 Elastic SIEM (EQL — Event Query Language)
```eql
// Process injection detection
sequence by host.name
  [process where process.name == "cmd.exe"]
  [network where destination.port == 4444]

// Detect Mimikatz
process where process.name == "lsass.exe"
  and process.parent.name != "wininit.exe"
```

---

## 6. SIEM Alert Lifecycle

```
Alert Generated by correlation rule
        │
        ▼
Tier 1 SOC Analyst — Initial Triage
  ├── Is this a known false positive? → Document, close, tune rule
  ├── Can I confirm malicious activity? → Escalate to Tier 2
  └── Need more context → Investigate further
        │
        ▼
Tier 2 SOC Analyst — Deep Investigation
  ├── Pull full logs, packet captures, endpoint data
  ├── Determine scope — what was accessed, what was exfiltrated?
  ├── Identify attack vector and timeline
  └── True incident confirmed → Escalate to Tier 3 / IR team
        │
        ▼
Tier 3 / Incident Response Team
  ├── Containment — isolate affected systems
  ├── Eradication — remove malware, close access
  ├── Recovery — restore systems, validate clean
  └── Lessons learned → Update SIEM rules
```

---

## 7. SIEM Tuning — Reducing False Positives

```
Problem: Too many false positives → alert fatigue → real threats missed

Tuning strategies:
  ├── Whitelist known-good IPs / users / processes
  ├── Adjust thresholds (raise brute force from 5 → 10 attempts)
  ├── Add context filters (only alert if src_ip is external)
  ├── Time-based suppression (suppress during maintenance windows)
  ├── Risk scoring (only alert if risk score > threshold)
  └── Correlation tuning (require multiple conditions before alerting)

Goal: High fidelity alerts — every alert worth investigating
```

---

## 8. Popular SIEM Platforms

| Platform | Developer | Deployment | Notes |
|----------|-----------|-----------|-------|
| **Splunk** | Splunk Inc. | On-prem / Cloud | Industry leader, powerful SPL, expensive |
| **Microsoft Sentinel** | Microsoft | Cloud-native (Azure) | Great for Microsoft environments, KQL |
| **IBM QRadar** | IBM | On-prem / Cloud | Enterprise, complex, rule-based |
| **Elastic SIEM** | Elastic | On-prem / Cloud | Open source core, scalable |
| **Google Chronicle** | Google | Cloud-native | Petabyte scale, YARA-L rules |
| **LogRhythm** | LogRhythm | On-prem / Cloud | SMB to enterprise |
| **ArcSight** | Micro Focus | On-prem | Legacy enterprise SIEM |
| **Wazuh** | Open source | On-prem | Free, great for home labs / learning |

> **For learning:** Wazuh (free, open source) and Elastic SIEM (free tier) are excellent starting points. TryHackMe and Hack The Box have Splunk and Sentinel labs.

---

## 9. Key Terms & Terminology

| Term | Meaning |
|------|---------|
| **SIEM** | Security Information and Event Management |
| **SIM** | Security Information Management — log storage and search |
| **SEM** | Security Event Management — real-time correlation and alerting |
| **Normalisation** | Converting diverse log formats to common schema |
| **Correlation** | Combining multiple events to detect complex threats |
| **Alert** | Notification generated when a rule fires |
| **True positive** | Real threat correctly detected |
| **False positive** | Legitimate activity incorrectly flagged |
| **Alert fatigue** | Analysts overwhelmed by too many false positives |
| **Tuning** | Adjusting rules to reduce false positives |
| **IOC** | Indicator of Compromise — IP, domain, hash, URL |
| **TTP** | Tactics, Techniques, Procedures — attacker methods |
| **UEBA** | User and Entity Behaviour Analytics |
| **SOAR** | Security Orchestration, Automation and Response — automates SIEM responses |
| **Retention** | How long logs are stored |
| **SPL** | Splunk Processing Language — Splunk query language |
| **KQL** | Kusto Query Language — Microsoft Sentinel |
| **MITRE ATT&CK** | Framework mapping attacker TTPs to detection opportunities |
| **Use case** | Specific detection scenario implemented as SIEM rule |
| **Playbook** | Step-by-step response procedure for a specific alert type |

---

## 10. 🔐 SOC Analyst Daily SIEM Workflow

```
Start of shift:
  1. Check alert queue — how many open alerts?
  2. Review overnight alerts — anything critical missed?
  3. Check dashboards — any anomalies in overnight traffic?

Alert triage:
  4. Open highest severity alert
  5. Read alert description — what triggered it?
  6. Query SIEM for context:
     - What else did this IP/user/host do?
     - Is this a known pattern?
     - Any related alerts?
  7. Determine: TP or FP?
  8. Document findings in ticketing system

Investigation (if TP):
  9. Expand scope — what else was affected?
  10. Build attack timeline
  11. Identify IOCs — extract IPs, domains, hashes
  12. Escalate with full context to Tier 2

End of shift:
  13. Hand over open investigations
  14. Document any new false positive patterns for tuning
  15. Update threat intel with new IOCs discovered
```

---

## 11. Common Interview Questions

**Q1. What is a SIEM and what are its core functions?**
> A SIEM collects logs from across the entire IT environment, normalises them into a common format, correlates events using detection rules, and generates alerts when threats are detected. Core functions: centralised log storage and search (SIM), real-time event correlation and alerting (SEM), threat intelligence integration, compliance reporting, and providing the primary investigation interface for SOC analysts.

**Q2. What is the difference between a true positive and a false positive?**
> A true positive is when the SIEM correctly identifies real malicious activity. A false positive is when legitimate activity is incorrectly flagged as malicious. False positives are a major operational challenge — too many cause alert fatigue, where analysts become desensitised and may miss real threats buried in noise. SIEM tuning reduces false positives to improve signal quality.

**Q3. What is log normalisation and why is it important?**
> Log normalisation converts log data from hundreds of different sources and formats (syslog, JSON, XML, CEF, LEEF) into a common, consistent schema. Without normalisation, correlating events across a firewall log and a Windows event log would be impossible — they use completely different field names and formats. Normalisation enables cross-source correlation and consistent querying.

**Q4. What is UEBA and how does it improve SIEM detection?**
> UEBA (User and Entity Behaviour Analytics) uses machine learning to profile the normal behaviour of each user and entity, then assigns risk scores to deviations from that baseline. It improves SIEM detection by catching threats that rule-based detection misses — like a compromised account used at normal times and locations but accessing unusual data volumes.

**Q5. Walk me through how you would investigate a brute force alert in a SIEM.**
> First I'd confirm the alert details — source IP, targeted accounts, timeframe. I'd query SIEM for all login events from that IP to see the full scope. Check if any login succeeded after the failures — that's the critical question. If yes, I'd investigate what the account did after the successful login, check for lateral movement, data access, and privilege escalation. I'd also check threat intel feeds for the source IP. If confirmed compromise, escalate to incident response.

**Q6. What Windows Event IDs are most important for a SOC analyst?**
> The most critical: 4624 (successful logon), 4625 (failed logon — brute force), 4648 (explicit credential use — lateral movement), 4672 (special privileges — escalation), 4698 (scheduled task — persistence), 4720 (new user account — backdoor), 4732 (added to admin group — escalation), and 1102 (audit log cleared — covering tracks).

---

## 12. Quick Revision Summary

```
SIEM = SIM (storage + search) + SEM (real-time correlation + alerting)

SIEM workflow:
  Collect → Normalise → Index → Correlate → Alert → Investigate → Respond

Detection methods:
  Rule-based      →  Predefined conditions (brute force, known ports)
  Anomaly-based   →  Deviation from baseline (impossible travel)
  Threat intel    →  Match against known IOCs
  UEBA            →  ML-based user/entity behaviour scoring

Critical Windows Event IDs:
  4624  →  Successful login
  4625  →  Failed login (brute force)
  4648  →  Explicit credentials (lateral movement)
  4672  →  Special privileges (escalation)
  4698  →  Scheduled task (persistence)
  4720  →  New user account (backdoor)
  1102  →  Log cleared (cover tracks)

Popular SIEMs:
  Splunk          →  Industry leader, SPL query language
  Microsoft Sentinel → Cloud-native, KQL, Azure-integrated
  IBM QRadar      →  Enterprise, rule-based
  Elastic SIEM    →  Open source, scalable
  Wazuh           →  Free, great for learning

SOC tiers:
  Tier 1  →  Alert triage, initial investigation
  Tier 2  →  Deep investigation, scope determination
  Tier 3  →  Incident response, forensics, remediation

Key concepts:
  Alert fatigue   →  Too many FPs → analysts miss real threats
  Tuning          →  Reduce FPs → improve signal quality
  SOAR            →  Automate response to common alert types
  Playbook        →  Step-by-step response procedure per alert type
```

---

*Previous topic → [Network Monitoring](./notes.md)*
*Next topic → [Incident Response Fundamentals](./incident-response.md)*
