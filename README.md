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

📎 [LinkedIn](https://www.linkedin.com/in/lereko-mohlomi/)

---

## Labs Index

### CompTIA Security+ — CertMaster Labs

**Penetration Testing**

| # | Lab | Technique | Status |
|---|-----|-----------|--------|
| 01 | [Directory Traversal](./security-plus/penetration-testing/01-directory-traversal.md) | Path Traversal / LFI | ✅ Complete |
| 02 | [Command Injection](./security-plus/penetration-testing/02-command-injection.md) | OS Command Injection | ✅ Complete |
| 03 | [File Upload & Reverse Shell](./security-plus/penetration-testing/03-file-upload-webshell.md) | RCE via File Upload + Metasploit | ✅ Complete |

**Vulnerability Management**

| # | Lab | Technique | Status |
|---|-----|-----------|--------|
| 04 | [SQL Injection Exploitation](./security-plus/vulnerability-management/01-sql-injection-exploitation.md) | Boolean Logic, UNION-based Extraction, Schema Enumeration | ✅ Complete |
| 05 | [SQLi Log Investigation](./security-plus/vulnerability-management/02-sqli-log-investigation.md) | Apache Log Analysis, IoC Identification, Percent-Encoding | ✅ Complete |
| 06 | [Threat Feeds & Exploit Database](./security-plus/vulnerability-management/03-threat-feeds-exploit-db.md) | Threat Intelligence, AlienVault OTX, Exploit-DB, GHDB | ✅ Complete |
| 07 | [Vulnerability Scanning (GSA/OpenVAS)](./security-plus/vulnerability-management/04-vulnerability-scanning.md) | Credentialed Scanning, CVE/NVD, CVSS v3.1 | ✅ Complete |

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
cybersecurity-labs/
├── README.md
└── security-plus/
    ├── penetration-testing/
    |   |── README.md 
    │   ├── 01-directory-traversal.md
    │   ├── 02-command-injection.md
    │   └── 03-file-upload-webshell.md
    ├── vulnerability-management/
    |   |── README.md                        
    |   ├── 01-sql-injection-exploitation.md
    |   ├── 02-sqli-log-investigation.md
    |   ├── 03-threat-feeds-exploit-db.md
    |   └── 04-vulnerability-scanning.md
```

---

*This repository is actively maintained. New write-ups are added as labs are completed.*
