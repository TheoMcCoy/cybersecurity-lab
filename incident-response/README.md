# Incident Response

**Source:** CompTIA Security+ CertMaster Labs (Sections 12.1.11)
**Status:** ✅ Complete

---

## Overview

This folder contains write-ups from hands-on incident response, monitoring, and data source labs completed as part of the CompTIA Security+ CertMaster Lab pathway. Labs cover the full incident response lifecycle — from real-time SIEM-based detection through root cause analysis and multi-system breach investigation.

---

## Labs in This Folder

| # | Lab | Technique | Status |
|---|-----|-----------|--------|
| 01 | [Detecting Logon Events with Wazuh](./01-detecting-logon-events-wazuh.md) | Hydra brute-force, SMB mounting, SIEM alert review | ✅ Complete |
| 02 | [Detecting Anti-Forensics with Wazuh](./02-detecting-anti-forensics-wazuh.md) | Audit log clearing, anti-forensics detection (Rule 63103) | ✅ Complete |
| 03 | [Root Cause Analysis: Security Alert](./03-root-cause-analysis-security-alert.md) | Audit policy tampering, RDP lateral movement, Event Viewer correlation | ✅ Complete |
| 04 | [Root Cause Analysis: Full Breach Investigation](./04-root-cause-analysis-breach-investigation.md) | Phishing → proxy hijack → credential theft → RDP → audit tampering | ✅ Complete |
| 05 | [Network Sniffing with Wireshark](./05-network-sniffing-wireshark.md) | Packet capture, display filters, TCP/HTTP stream analysis | ✅ Complete |

---

## Tools Used

| Tool | Purpose |
|------|---------|
| Wazuh | SIEM/XDR — alert detection and review |
| Hydra | Dictionary-based password guessing simulation |
| Windows Event Viewer | Event log correlation and logon type analysis |
| auditpol | Audit policy state verification |
| OPNsense firewall | Network firewall log analysis |
| Mozilla Thunderbird | Email client — phishing email investigation |
| Wireshark | Packet capture and network traffic analysis |

---

## Key Concepts Covered

- SIEM alert levels, rule components, and MITRE ATT&CK mapping
- Wazuh Rule IDs: 92652 (successful password discovery), 60122 (logon failure), 60106 (logon success), 63103 (audit log cleared), 60112 (audit policy change)
- Windows logon types (2, 3, 7, 8, 9, 10, 11)
- Anti-forensics detection — audit log clearing (T1070.001)
- Root cause analysis methodology — following the evidence chain across multiple systems
- Full attack chain reconstruction: phishing → proxyset.bat → proxy hijack → credential interception → RDP → audit tampering
- Wireshark display filters (ip.dst, ip.ttl, ARP, TCP flags, NOT logic)
- Follow TCP Stream vs Follow HTTP Stream — and the HTTPS limitation

---

*Write-ups by Lereko Mohlomi | [LinkedIn](https://www.linkedin.com/in/lereko-mohlomi/) | [Back to Main Repo](../../README.md)*
