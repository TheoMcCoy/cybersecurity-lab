# Lab 02 — Detecting Anti-Forensics with Wazuh

**Source:** CompTIA Security+ CertMaster Labs — Incident Response: Detection (Section 12.1.11)
**Environment:** DC10 (Windows Server 2019) → WAZUH (Ubuntu Server) → KALI
**Tool:** Wazuh SIEM/XDR platform, Windows Event Viewer
**Technique:** Anti-forensics — Windows audit log clearing
**MITRE ATT&CK:** T1070.001 — Indicator Removal: Clear Windows Event Logs
**Security+ Objectives:** 4.4 / 4.8 / 4.9
**Status:** ✅ Complete

---

## Scenario

Anti-forensics are activities performed by intruders to mask, hide, or destroy evidence of their malicious actions on a system. One of the most common anti-forensic techniques is clearing Windows event logs — an attacker who has gained elevated privileges will often delete audit logs immediately after completing their objective to prevent investigators from reconstructing the attack. This exercise demonstrates how wazuh detects this IoC even after the logs themselves have been destroyed.

---

## Environment

| System | Role | IP |
|--------|------|----|
| DC10 | Domain controller — target (Windows Server 2019) | 10.1.16.1 |
| WAZUH | SIEM platform (Ubuntu Server) | 10.1.16.242 |
| KALI | Security workstation | 10.1.16.66 |

---

## Part 1 — Simulate Anti-Forensic Activity: Clear the Security Log

Connected to DC10 as `Structureality\Administrator` (Pa$$w0rd).

Cleared the Windows Security event log via Event Viewer:

1. Opened **Event Viewer** (search → Event → Event Viewer)
2. Expanded **Windows Logs** in the left pane
3. Selected **Security**
4. Selected **Clear log…** from the right pane
5. Confirmed **Clear** on the confirmation dialog
6. Closed Event Viewer

**Effect:** All previously recorded Security log entries on DC10 are now deleted. From an attacker's perspective, this eliminates the audit trail of everything they did while elevated on this system.

---

## Part 2 — Detect the Anti-Forensic Activity in Wazuh

Returned to KALI and accessed the wazuh dashboard at `https://10.1.16.242`.

Navigated to **Security events** and searched for Rule ID **63103**.

**Alert detected:**

| Field | Value |
|-------|-------|
| Rule ID | 63103 |
| Description | The audit log was cleared |
| Level | (medium-high — audit log clearing is a significant IoC) |

> Scored question answered correctly: Description is **"The audit log was cleared"** (not "A log file was cleared", "Event viewer logs were cleared", or "A Windows log file was cleared")

---

## Why This Matters

The attacker cleared the log on DC10 — but they cannot clear the wazuh record on the separate SIEM server without also compromising WAZUH itself. This is exactly why forwarding logs to a **centralised, hardened SIEM** is a critical security control:

| Scenario | Evidence preserved? |
|----------|-------------------|
| Attacker clears Windows Security log | ❌ Local log destroyed |
| Wazuh agent had already forwarded events | ✅ SIEM record intact |
| Attacker also compromises WAZUH | ❌ Only if WAZUH not write-protected |

The window between when an event occurs and when it is forwarded to the SIEM is the attacker's only opportunity to prevent detection. High-frequency log forwarding (near real-time) minimises this window.

---

## Defensive Recommendations

| Control | Description |
|---------|-------------|
| Centralised SIEM | Forward all Windows event logs to an external, hardened log server in real time |
| Alert on log clearing | Rule 63103 and similar rules should trigger immediate high-priority alerts |
| Restrict log clearing | Use GPO to prevent non-administrative accounts from clearing event logs |
| Write-once log storage | Use append-only or WORM storage for critical audit logs |
| Monitor WAZUH access | Alert on any authentication to the SIEM server itself — a compromised SIEM is a blind spot |

---

## Key Takeaway

Log clearing is one of the first things an experienced attacker does after achieving their objective — but it only works if the logs were never forwarded. Wazuh detected the clearing event (Rule 63103) because the wazuh agent on DC10 had already streamed the relevant events to the centralised server. **The lesson: the value of a SIEM is proportional to how quickly it receives and retains log data.** A SIEM that polls logs every hour provides a one-hour window in which an attacker can operate and cover their tracks.

---

*Write-up by Lereko Mohlomi | [LinkedIn](https://www.linkedin.com/in/lereko-mohlomi/) | [Back to Incident Response](./README.md)*
