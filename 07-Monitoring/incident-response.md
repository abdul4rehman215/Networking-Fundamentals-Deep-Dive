 # 🚨 Incident Response Fundamentals — Full Deep Dive

> **Source:** Professor Messer | **Topic #:** 18 | **Module:** 3 | **Status:** ✅ Complete

---

## 1. Definition

**Incident Response (IR)** is the structured, systematic process an organisation follows to prepare for, detect, contain, eradicate, and recover from cybersecurity incidents — minimising damage, reducing recovery time, and preventing recurrence.

**What is a security incident?**
```
An incident = any event that threatens the:
  ├── Confidentiality — unauthorised access to data
  ├── Integrity — unauthorised modification of data
  └── Availability — disruption of systems or services

Examples:
  ├── Ransomware infection encrypting files
  ├── Data breach — customer PII exfiltrated
  ├── DDoS attack taking down web services
  ├── Insider threat — employee stealing data
  ├── Phishing campaign compromising credentials
  └── APT (Advanced Persistent Threat) — long-term network intrusion
```

> A security incident without an IR plan is like a hospital without emergency procedures — chaos, delayed response, and worse outcomes. IR turns crisis into a controlled process.

---

## 2. NIST Incident Response Framework

The **NIST SP 800-61** (Computer Security Incident Handling Guide) defines the gold standard IR lifecycle — 4 phases (with Containment/Eradication/Recovery combined as one):

```
┌─────────────────────────────────────────────────────────┐
│                    IR LIFECYCLE                          │
│                                                          │
│  ┌──────────┐    ┌──────────────────┐    ┌───────────┐  │
│  │          │    │                  │    │           │  │
│  │  Prep    │───►│ Detection &      │───►│Containment│  │
│  │          │    │ Analysis         │    │Eradication│  │
│  └──────────┘    └──────────────────┘    │ Recovery  │  │
│       ▲                                  └─────┬─────┘  │
│       │                                        │        │
│  ┌────┴───────────────────────────────────┐    │        │
│  │         Post-Incident Activity          │◄───┘        │
│  │         (Lessons Learned)              │             │
│  └─────────────────────────────────────────┘            │
└─────────────────────────────────────────────────────────┘
```

---

## 3. The 6 Phases — Deep Dive

### Phase 1 — Preparation
**Goal: Be ready before an incident occurs**

```
Preparation activities:
  ├── Develop IR policy and plan
  ├── Build and train the IR team (CSIRT)
  ├── Define roles and responsibilities
  ├── Deploy detection and monitoring tools (SIEM, IDS, EDR)
  ├── Establish communication procedures
  │   ├── Who gets notified and when?
  │   ├── How to communicate securely during incident?
  │   └── Legal/PR contacts for data breach notification
  ├── Create playbooks for common incident types
  │   ├── Ransomware playbook
  │   ├── Phishing playbook
  │   ├── Data breach playbook
  │   └── DDoS playbook
  ├── Establish out-of-band communication (if email is compromised)
  ├── Inventory critical assets — know what needs protecting most
  ├── Define escalation thresholds
  └── Conduct tabletop exercises and simulations
```

**Key IR team roles:**
| Role | Responsibility |
|------|---------------|
| **IR Manager** | Coordinates overall response, decision maker |
| **SOC Analyst (Tier 1)** | Alert triage, initial detection |
| **IR Analyst (Tier 2/3)** | Deep investigation, forensics |
| **Threat Intel Analyst** | IOC enrichment, attacker attribution |
| **Legal counsel** | Regulatory compliance, breach notification |
| **PR/Communications** | External communications, media |
| **IT/Sysadmin** | System isolation, restoration |
| **Management/CISO** | Executive decisions, business continuity |

---

### Phase 2 — Detection & Analysis
**Goal: Confirm an incident has occurred and understand its scope**

```
Detection sources:
  ├── SIEM alerts
  ├── IDS/IPS signatures
  ├── EDR/AV detections
  ├── User reports ("I can't access my files")
  ├── Threat intelligence feeds
  ├── External notification (FBI, CISA, partner organisation)
  └── Penetration test / bug bounty findings
```

**Initial triage questions:**
```
1. Is this a real incident or a false positive?
2. What systems are affected?
3. When did it start? (Patient zero identification)
4. How did the attacker gain initial access?
5. Is the attacker still active?
6. What data may have been accessed or exfiltrated?
7. What is the business impact?
8. What is the severity level?
```

**Incident severity classification:**
| Severity | Description | Response time |
|----------|-------------|---------------|
| **Critical (P1)** | Active breach, ransomware, data exfiltration in progress | Immediate — all hands |
| **High (P2)** | Confirmed compromise, potential data exposure | Within 1 hour |
| **Medium (P3)** | Suspicious activity, unconfirmed compromise | Within 4 hours |
| **Low (P4)** | Policy violation, minor anomaly | Within 24 hours |

**Evidence collection:**
```
Volatile evidence (collect FIRST — lost on reboot):
  ├── Running processes (tasklist, ps aux)
  ├── Network connections (netstat -ano)
  ├── Logged-in users (who, query user)
  ├── Memory dump (for malware analysis)
  └── Clipboard contents

Non-volatile evidence (survives reboot):
  ├── Disk image / forensic copy
  ├── Log files (SIEM, Windows Event Log, syslog)
  ├── Registry hives (Windows)
  ├── Browser history and artifacts
  └── Email headers
```

**Chain of custody:**
```
Every piece of evidence must be:
  ├── Documented — what it is, when collected, by whom
  ├── Hashed — MD5/SHA256 to prove it hasn't been modified
  ├── Stored securely — tamper-evident
  └── Tracked — every person who handled it logged
```

---

### Phase 3 — Containment
**Goal: Stop the spread without destroying evidence**

**Short-term containment (immediate):**
```
├── Isolate infected host from network (VLAN isolation or unplug)
├── Block attacker IPs at firewall
├── Disable compromised accounts
├── Reset compromised credentials
├── Block malicious domains at DNS
└── Take memory dump before isolation (preserve volatile evidence)
```

**Long-term containment (stabilise):**
```
├── Apply emergency patches
├── Increase monitoring on adjacent systems
├── Implement additional authentication (MFA enforcement)
├── Deploy honeypots to detect attacker pivoting
└── Maintain business operations with clean systems
```

**Containment strategies:**
| Strategy | When to use | Trade-off |
|----------|-------------|-----------|
| **Full isolation** | Ransomware, active exfiltration | Disrupts business |
| **VLAN isolation** | Investigate while containing | Limited visibility |
| **Traffic blocking** | Block specific IPs/domains | May not stop all C2 |
| **Account disable** | Compromised credentials | User disruption |
| **Sinkholing** | Redirect C2 traffic to analyst-controlled server | Intelligence gathering |

---

### Phase 4 — Eradication
**Goal: Completely remove the threat from the environment**

```
Eradication steps:
  ├── Identify all affected systems (not just patient zero)
  ├── Remove malware:
  │   ├── Automated removal via EDR/AV
  │   └── Manual removal if sophisticated malware
  ├── Close all backdoors and persistence mechanisms:
  │   ├── Scheduled tasks
  │   ├── Registry run keys
  │   ├── New user accounts created by attacker
  │   ├── SSH authorized_keys modified
  │   └── Web shells on servers
  ├── Reset ALL potentially compromised credentials
  ├── Patch the vulnerability that allowed initial access
  ├── Verify eradication with fresh scans
  └── Document all artefacts removed
```

**Persistence mechanisms to check (Windows):**
```
Registry:
  HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Run
  HKCU\SOFTWARE\Microsoft\Windows\CurrentVersion\Run

Scheduled tasks:
  schtasks /query /fo LIST /v

Services:
  sc query | Event ID 7045

Startup folders:
  C:\Users\[user]\AppData\Roaming\Microsoft\Windows\Start Menu\Programs\Startup

WMI subscriptions:
  Get-WMIObject -Namespace root\subscription -Class __EventFilter
```

---

### Phase 5 — Recovery
**Goal: Restore systems to normal operation safely**

```
Recovery steps:
  ├── Restore from clean, verified backups
  │   (Verify backup integrity BEFORE restoring)
  ├── Rebuild compromised systems from scratch (if heavily infected)
  ├── Re-image endpoints (guaranteed clean state)
  ├── Validate system integrity before returning to production
  ├── Gradually restore connectivity with enhanced monitoring
  ├── Monitor closely for signs of reinfection:
  │   ├── Same IOCs reappearing
  │   ├── Same C2 domains queried
  │   └── Same persistence mechanisms recreated
  └── Confirm business operations are fully restored
```

**Recovery decision — restore vs rebuild:**
```
Restore from backup:
  ├── Faster recovery
  ├── Risk: backup may contain malware if taken after infection
  └── Verify backup date predates infection

Rebuild from scratch:
  ├── Guaranteed clean state
  ├── Slower — more effort
  └── Preferred for critical servers or heavily compromised systems
```

---

### Phase 6 — Lessons Learned (Post-Incident Activity)
**Goal: Improve future response and prevent recurrence**

```
Post-incident review (within 2 weeks of resolution):

Questions to answer:
  ├── What happened exactly? (full timeline)
  ├── When was the incident first detected?
  ├── Could it have been detected earlier?
  ├── What was the root cause?
  ├── What was the business impact?
  ├── What worked well in the response?
  ├── What didn't work well?
  ├── What would we do differently?
  └── What changes do we need to prevent recurrence?

Outputs:
  ├── Incident report (executive summary + technical detail)
  ├── Updated SIEM detection rules
  ├── Patched vulnerabilities
  ├── Revised security policies
  ├── Updated IR playbooks
  ├── New security controls implemented
  └── Training identified for team
```

---

## 4. Incident Classification

### 4.1 By Type

| Incident type | Description |
|---------------|-------------|
| **Malware** | Virus, worm, trojan, ransomware, spyware |
| **Phishing** | Email-based credential theft or malware delivery |
| **Unauthorised access** | Account compromise, insider threat |
| **Data breach** | Exfiltration of sensitive data |
| **DDoS** | Distributed Denial of Service — availability attack |
| **Web application attack** | SQLi, XSS, web shell upload |
| **APT** | Advanced Persistent Threat — long-term stealth intrusion |
| **Insider threat** | Malicious or negligent employee action |
| **Supply chain attack** | Compromise via third-party software/vendor |

### 4.2 Attack Lifecycle — Cyber Kill Chain (Lockheed Martin)

```
1. Reconnaissance    →  Attacker researches target (OSINT, scanning)
2. Weaponisation     →  Creates exploit/payload (malicious doc, dropper)
3. Delivery          →  Sends payload (phishing email, watering hole)
4. Exploitation      →  Payload executes, vulnerability triggered
5. Installation      →  Malware installed, persistence established
6. C2               →  Attacker establishes command and control channel
7. Actions on obj.   →  Attacker achieves goal (exfil, encrypt, destroy)

IR goal: Detect and stop as early in the chain as possible
  Detection at step 3 (delivery) = much better than step 7 (exfiltration)
```

### 4.3 MITRE ATT&CK Framework

```
Organises attacker TTPs into:
  ├── Tactics (14) — the "why" — what the attacker is trying to achieve
  │   Examples: Initial Access, Execution, Persistence, Privilege Escalation,
  │             Defense Evasion, Credential Access, Discovery, Lateral Movement,
  │             Collection, C2, Exfiltration, Impact
  │
  └── Techniques (hundreds) — the "how" — specific methods used
      Example: T1566 Phishing, T1053 Scheduled Task, T1059 Command Interpreter

SOC use: Map detected activity to ATT&CK → understand attacker's stage
         → predict next moves → hunt for remaining TTPs
```

---

## 5. IR Documentation

### 5.1 Incident Ticket Fields
```
Incident ID:
Date/Time detected:
Date/Time reported:
Reported by:
Severity: Critical / High / Medium / Low
Status: Open / Investigating / Contained / Resolved / Closed
Affected systems:
Affected users:
Attack vector:
IOCs (IPs, domains, hashes, filenames):
Timeline of events:
Actions taken:
Current status:
Next steps:
Assigned to:
Escalated to:
```

### 5.2 Incident Timeline Format
```
[2026-05-20 02:14:22] SIEM alert fires — brute force against VPN from 45.33.32.156
[2026-05-20 02:15:01] Analyst opens ticket, begins triage
[2026-05-20 02:16:45] Confirmed — 847 failed logins, then 1 success at 02:14:22
[2026-05-20 02:17:30] Account "m.zafar" successfully authenticated from Germany
[2026-05-20 02:18:00] Account disabled immediately
[2026-05-20 02:19:15] VPN session terminated
[2026-05-20 02:20:00] Reviewed what account accessed during 5-minute window
[2026-05-20 02:25:00] No lateral movement detected — contained to VPN access only
[2026-05-20 02:30:00] Password reset, MFA enrolled, account re-enabled
[2026-05-20 02:35:00] Source IP blocked at firewall
[2026-05-20 08:00:00] User contacted — confirmed account compromise, not user action
[2026-05-20 09:00:00] Escalated to threat intel — IP linked to known credential stuffing campaign
[2026-05-20 14:00:00] Incident closed — no data exposure confirmed
```

---

## 6. Key Terms & Terminology

| Term | Meaning |
|------|---------|
| **IR** | Incident Response — structured process for handling security incidents |
| **CSIRT** | Computer Security Incident Response Team |
| **CIRT** | Computer Incident Response Team |
| **SOC** | Security Operations Centre — monitors and responds to incidents |
| **Triage** | Initial assessment to determine severity and priority |
| **Containment** | Stopping the spread of an incident |
| **Eradication** | Completely removing the threat |
| **Recovery** | Restoring systems to normal operation |
| **Chain of custody** | Documentation proving evidence integrity |
| **IOC** | Indicator of Compromise — evidence of malicious activity |
| **TTP** | Tactics, Techniques, Procedures — attacker methods |
| **Kill chain** | Model of attacker progression through an attack |
| **MITRE ATT&CK** | Framework mapping attacker TTPs |
| **Playbook** | Step-by-step procedure for a specific incident type |
| **Tabletop exercise** | Simulated incident discussion to test IR plan |
| **Patient zero** | First infected system in an incident |
| **Forensic image** | Bit-for-bit copy of storage for evidence |
| **Volatile data** | Data lost when system powers off (RAM, connections) |
| **Sinkhole** | Redirect malicious traffic to analyst-controlled server |
| **NIST SP 800-61** | NIST guide for computer security incident handling |

---

## 7. 🔐 SOC Analyst Role in IR

```
Tier 1 SOC Analyst (first responder):
  ├── Monitor SIEM alert queue
  ├── Initial triage — TP or FP?
  ├── Document initial findings in ticket
  ├── Notify Tier 2 if confirmed incident
  └── Basic containment (block IP, disable account)

Tier 2 SOC Analyst (investigator):
  ├── Deep dive investigation
  ├── Timeline reconstruction
  ├── Scope determination — what was affected?
  ├── IOC extraction and threat intel enrichment
  ├── Recommend containment and eradication steps
  └── Escalate to Tier 3 / IR team if major incident

Tier 3 / IR Specialist (responder):
  ├── Full forensic investigation
  ├── Memory and disk analysis
  ├── Malware reverse engineering
  ├── Legal and compliance coordination
  ├── Executive reporting
  └── Post-incident review leadership
```

### SIEM Queries During IR

```
# Identify all systems attacker touched (lateral movement)
index=auth src_ip=45.33.32.156 OR username="compromised_user"
| stats values(dest_host) as systems_accessed by username

# Find all IOCs related to incident
index=* (src_ip=45.33.32.156 OR dst_domain="evil-c2.com" OR file_hash="abc123...")
| table timestamp, host, src_ip, dst_ip, process, file_hash

# Reconstruct full timeline
index=* username="compromised_user" earliest="2026-05-20T02:00:00"
| sort timestamp
| table timestamp, host, event_type, src_ip, action

# Check for persistence mechanisms after compromise
index=windows EventCode IN (4698, 7045) earliest="2026-05-20T02:00:00"
| table timestamp, host, task_name, service_name, username

# Hunt for same IOCs across entire environment
index=* dst_ip IN (known_c2_ips) OR dns_query IN (known_c2_domains)
| stats count by host | sort -count
```

---

## 8. Common Interview Questions

**Q1. What are the 6 phases of incident response?**
> Preparation (build IR plan, tools, team before incidents), Detection & Analysis (confirm incident, determine scope), Containment (stop the spread — short and long term), Eradication (completely remove the threat), Recovery (restore systems to normal operation), and Lessons Learned (post-incident review to improve future response).

**Q2. What is the difference between containment and eradication?**
> Containment stops the incident from spreading further — isolating affected systems, blocking attacker IPs, disabling compromised accounts — while the threat may still exist on affected systems. Eradication completely removes the threat — deleting malware, closing backdoors, removing persistence mechanisms, resetting all compromised credentials. Containment is immediate; eradication is thorough.

**Q3. Why is the "Lessons Learned" phase important?**
> Without lessons learned, the same incident will happen again. This phase identifies the root cause, what the IR team did well and poorly, what controls failed, and what improvements are needed. It drives updates to SIEM detection rules, IR playbooks, security policies, and team training — making the organisation genuinely stronger after every incident.

**Q4. What is the Cyber Kill Chain and how does it help SOC analysts?**
> The Cyber Kill Chain is Lockheed Martin's model of attacker progression through 7 stages — Reconnaissance, Weaponisation, Delivery, Exploitation, Installation, C2, and Actions on Objectives. It helps SOC analysts understand where in the attack lifecycle a detected activity sits, predict what the attacker will do next, and hunt for evidence of the remaining stages. The earlier detection occurs in the chain, the less damage the attacker causes.

**Q5. What is chain of custody and why does it matter?**
> Chain of custody documents every piece of evidence collected during an incident — what it is, when it was collected, by whom, and who has handled it since. It matters because if an incident leads to legal action, evidence without proper chain of custody can be inadmissible in court. It also proves evidence integrity — that it hasn't been tampered with since collection.

**Q6. What would you do in the first 15 minutes of a ransomware incident?**
> First, confirm it's ransomware — check for encrypted files, ransom note, SIEM alerts. Immediately notify the IR team and management. Isolate affected systems from the network to prevent spread — but take memory dumps first to preserve volatile evidence. Block known ransomware C2 IPs and domains at the firewall. Identify patient zero — which system was first infected and how. Check if backups are accessible and unaffected. Begin documenting everything. Do NOT pay the ransom without legal and management approval.

---

## 9. Quick Revision Summary

```
IR phases (NIST SP 800-61):
  1. Preparation        →  Plan, train, deploy tools BEFORE incident
  2. Detection/Analysis →  Confirm incident, scope, timeline, severity
  3. Containment        →  Stop spread (short-term) + stabilise (long-term)
  4. Eradication        →  Remove threat completely
  5. Recovery           →  Restore systems, monitor for reinfection
  6. Lessons Learned    →  Post-incident review, update defences

Severity levels:
  P1 Critical  →  Active breach, ransomware — immediate response
  P2 High      →  Confirmed compromise — within 1 hour
  P3 Medium    →  Suspicious activity — within 4 hours
  P4 Low       →  Minor anomaly — within 24 hours

Evidence types:
  Volatile     →  RAM, connections, processes — collect FIRST
  Non-volatile →  Disk, logs, registry — survives reboot

Cyber Kill Chain:
  Recon → Weaponise → Deliver → Exploit → Install → C2 → Act
  Goal: Detect as early as possible in the chain

MITRE ATT&CK:
  14 tactics mapped to hundreds of techniques
  Use to understand attacker stage and predict next moves

Key IR tools:
  SIEM      →  Detection, investigation, timeline
  EDR       →  Endpoint containment, process visibility
  Forensics →  Volatility (memory), Autopsy (disk), FTK
  Threat intel → IOC enrichment, attacker attribution

SOC tiers in IR:
  Tier 1  →  Alert triage, initial containment
  Tier 2  →  Deep investigation, scope, escalation
  Tier 3  →  Forensics, malware analysis, executive reporting
```

---

*Previous topic → [SIEM Basics](./siem.md)*

---

## 🎉 Module 3 Complete!

Topics covered in this module:
- ✅ Wireless Networking
- ✅ VPNs
- ✅ Network Monitoring
- ✅ SIEM Basics
- ✅ Incident Response Fundamentals

**All 3 modules complete — 18 topics documented!**
