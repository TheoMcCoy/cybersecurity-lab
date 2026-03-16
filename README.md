# cybersecurity-labs

A portfolio of hands-on cybersecurity labs, challenge activities, and practical write-ups by **Lereko Mohlomi**.

Each write-up documents real lab work — covering the scenario, methodology, tools used, findings, and defensive takeaways. These are not theory summaries; they are records of practical work completed in controlled environments.

---

## About Me

Cloud and cybersecurity professional with 10+ years of IT infrastructure experience, focused on AWS security engineering, web application security, and incident response.

- AWS Certified Solutions Architect – Associate
- AWS Cloud Practitioner
- CompTIA Security+ *(completed full CertMaster Learn + Labs pathway — 15 domains)*
- ISC² Certified in Cybersecurity (CC)
- CCNA Exploration: Routing Protocols and Concepts
- CCNA Exploration: Access the WAN

📎 [LinkedIn](https://www.linkedin.com/in/lereko-mohlomi/)

---

## Labs Index

### CompTIA Security+ — CertMaster Labs

**Penetration Testing**

| # | Lab | Technique | Status |
|---|-----|-----------|--------|
| 01 | [Directory Traversal](./penetration-testing/01-directory-traversal.md) | Path Traversal / LFI | ✅ Complete |
| 02 | [Command Injection](./penetration-testing/02-command-injection.md) | OS Command Injection | ✅ Complete |
| 03 | [File Upload & Reverse Shell](./penetration-testing/03-file-upload-webshell.md) | RCE via File Upload + Metasploit | ✅ Complete |

**Vulnerability Management**

| # | Lab | Technique | Status |
|---|-----|-----------|--------|
| 04 | [SQL Injection Exploitation](./vulnerability-management/01-sql-injection-exploitation.md) | Boolean Logic, UNION-based Extraction, Schema Enumeration | ✅ Complete |
| 05 | [SQLi Log Investigation](./vulnerability-management/02-sqli-log-investigation.md) | Apache Log Analysis, IoC Identification, Percent-Encoding | ✅ Complete |
| 06 | [Threat Feeds & Exploit Database](./vulnerability-management/03-threat-feeds-exploit-db.md) | Threat Intelligence, AlienVault OTX, Exploit-DB, GHDB | ✅ Complete |
| 07 | [Vulnerability Scanning (GSA/OpenVAS)](./vulnerability-management/04-vulnerability-scanning.md) | Credentialed Scanning, CVE/NVD, CVSS v3.1 | ✅ Complete |

**Incident Response & Detection**

| # | Lab | Technique | Status |
|---|-----|-----------|--------|
| 08 | [Detecting Logon Events with Wazuh](./incident-response/01-detecting-logon-events-wazuh.md) | Hydra Brute-Force, SMB Mounting, SIEM Alert Review | ✅ Complete |
| 09 | [Detecting Anti-Forensics with Wazuh](./incident-response/02-detecting-anti-forensics-wazuh.md) | Audit Log Clearing, Anti-Forensics Detection | ✅ Complete |
| 10 | [Root Cause Analysis: Security Alert](./incident-response/03-root-cause-analysis-security-alert.md) | Audit Policy Tampering, RDP Lateral Movement, Event Viewer | ✅ Complete |
| 11 | [Root Cause Analysis: Breach Investigation](./incident-response/04-root-cause-analysis-breach-investigation.md) | Phishing → Proxy Hijack → Credential Theft → RDP → Audit Tampering | ✅ Complete |
| 12 | [Network Sniffing with Wireshark](./incident-response/05-network-sniffing-wireshark.md) | Packet Capture, Display Filters, TCP/HTTP Stream Analysis | ✅ Complete |

**Digital Forensics**

| # | Lab | Technique | Status |
|---|-----|-----------|--------|
| 13 | [Forensic Drive Image Analysis](./digital-forensics/01-forensic-drive-image-hidden-partition.md) | fdisk, testdisk, fiwalk, fsstat, mmls, fls, istat (TSK) | ✅ Complete |
| 14 | [Recovering Deleted Files](./digital-forensics/02-recovering-deleted-files.md) | tsk_recover, NTFS Undelete | ✅ Complete |
| 15 | [File Carving](./digital-forensics/03-file-carving.md) | testdisk Carving, Boot Sector Rebuild, FAT16 Recovery | ✅ Complete |

---

### CompTIA Security+ — Graded Challenge Activities

> Proctored, unguided assessments requiring independent analysis and decision-making.

| # | Challenge | Scenario | Verified Badge | Status |
|---|-----------|---------|---------------|--------|
| 01 | Challenge Activity 1 | Social Engineering Detection & Incident Response | [🔗 View](https://badges.parchment.com/public/assertions/AwATXi4QQSSuvasiplN_zA?identity__email=ltheo.mohlomi%40gmail.com) | ✅ Complete |
| 02 | Challenge Activity 2 | DDoS & Brute-Force Investigation & Response | [🔗 View](https://badges.parchment.com/public/assertions/9xD4e49-R_ayr5iCq4hkRg?identity__email=ltheo.mohlomi%40gmail.com) | ✅ Complete |

---

### TryHackMe — Attacking and Defending AWS

| # | Lab | Technique | Status |
|---|-----|-----------|--------|
| 01 | IAM Privilege Escalation | AWS IAM Abuse | 🔄 In Progress |
| 02 | S3 Misconfiguration Exploitation | Exposed Bucket Enumeration | 🔄 Upcoming |
| 03 | Metadata Service Abuse (IMDSv1) | Credential Exfiltration | 🔄 Upcoming |
| 04 | CloudTrail Evasion & Detection | Log Analysis and Evasion | 🔄 Upcoming |

---

## Repository Structure

```
📁 cybersecurity-labs/
├── README.md
├── 📁 digital-forensics/
│   ├── README.md
│   ├── 01-forensic-drive-image-hidden-partition.md
│   ├── 02-recovering-deleted-files.md
│   ├── 03-file-carving.md
│   └── 📁 images/
│           fdisk.jpg
├── 📁 incident-response/
│   ├── README.md
│   ├── 01-detecting-logon-events-wazuh.md
│   ├── 02-detecting-anti-forensics-wazuh.md
│   ├── 03-root-cause-analysis-security-alert.md
│   ├── 04-root-cause-analysis-breach-investigation.md
│   ├── 05-network-sniffing-wireshark.md
│   └── 📁 images/
│           auditpol.jpg
│           opensense-firewall.jpg
│           opensense.jpg
│           proxy-batch-file.jpg
│           testdisk-file-listing.jpg
│           wazuh-rule-92653-92652.jpg
├── 📁 penetration-testing/
│   ├── README.md
│   ├── 01-directory-traversal.md
│   ├── 02-command-injection.md
│   ├── 03-file-upload-webshell.md
│   └── 📁 images/
│           metasploit.jpg
└── 📁 vulnerability-management/
    ├── README.md
    ├── 01-sql-injection-exploitation.md
    ├── 02-sqli-log-investigation.md
    ├── 03-threat-feeds-exploit-db.md
    └── 04-vulnerability-scanning.md
```

---

*This repository is actively maintained. New write-ups are added as labs are completed.*
