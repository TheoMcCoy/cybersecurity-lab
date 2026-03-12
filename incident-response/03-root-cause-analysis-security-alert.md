# Lab 03 — Root Cause Analysis: Investigating a Security Alert

**Source:** CompTIA Security+ CertMaster Labs — Alerting and Monitoring Tools (Section 12.1.11)
**Environment:** KALI → WAZUH → DC10 (Windows Server 2019) → MS10 (Windows Server 2016)
**Tools:** Wazuh SIEM, Windows Event Viewer, auditpol
**Technique:** Root cause analysis — audit policy tampering, RDP lateral movement
**MITRE ATT&CK:** T1562.002 — Impair Defenses: Disable Windows Event Logging / T1021.001 — RDP
**Security+ Objectives:** 2.2 / 2.4 / 4.4 / 4.8 / 4.9
**Status:** ✅ Complete

---

## Scenario

A security flash message from the SOC (Security Operations Center) indicated that several auditing policies on DC10 had been changed — a significant concern, as disabling auditing is a common anti-forensic tactic. As a security team member at Structureality Inc., the task was to investigate the cause of the alert, identify who made the changes, and determine how they accessed the system.

> This lab is simulated as occurring on 31 March 2023. Wazuh was configured to display data from that date range (March 31 @ 00:00:00 → April 1 @ 00:00:00).

---

## Environment

| System | Role | IP |
|--------|------|----|
| KALI | Security workstation | — |
| WAZUH | SIEM platform (Ubuntu Server) | 10.1.16.242 |
| DC10 | Domain controller (Windows Server 2019) | 10.1.16.1 |
| MS10 | Windows Server 2016 — origin of the RDP session | 10.1.16.2 |

---

## Part 1 — Investigate the Security Alert in Wazuh

### Step 1 — Identify the Alert

Navigated to **Security events** in wazuh, set the time range to **March 31, 2023 → April 1, 2023**, and searched for **Rule ID 60112**.

**Results:**

| Field | Value |
|-------|-------|
| Rule ID | 60112 |
| Alert count on DC10 | **17** |
| Associated username | **jaime** |

> Scored question answered correctly: 17 security alerts for Rule ID 60112 on DC10, username: jaime.

Each Rule ID 60112 alert showed:
- `data.win.eventdata.auditPolicyChanges`: **Success removed** and/or **Failure removed**

**Why this is suspicious:** Removing Success/Failure auditing from policies means future events of those types will no longer be recorded. An attacker with elevated privileges uses this to operate without leaving an audit trail.

### Step 2 — Identify the Connection Type

Searched wazuh for **jaime** to locate the logon event preceding the audit policy changes.

**Alert found:**

| Field | Value |
|-------|-------|
| Rule ID | 92653 |
| Connection type | **Remote Desktop Connection (RDP)** |
| Event Record ID | 17404 |
| Time of RDP alert | 17:05:26 |

> Scored question answered correctly: Remote Desktop Connection (RDP)

**Why this is suspicious:** The jaime account normally operates from PC10 (10.1.24.101) — their assigned workstation. The RDP connection came from **10.1.16.2**, which is MS10 — an older Windows Server 2016 system in the server subnet, not a standard user workstation. Establishing an RDP connection from a server to a domain controller, followed immediately by audit policy changes, is a significant red flag.

---

## Part 2 — Investigate the Breach on DC10

Switched to the DC10 system to verify the audit policy state directly.

### Step 3 — Confirm Audit Policies Were Disabled

From an elevated Command Prompt on DC10:

```cmd
auditpol /get /category:*
```

**Result:**

| Finding | Value |
|---------|-------|
| Audit policy status | **No Auditing** |

> Scored question answered correctly: No Auditing

This confirms that auditing was fully disabled across all categories on DC10. The wazuh alerts were accurate — the jaime account (or whoever used it) systematically stripped the auditing configuration.

### Step 4 — Correlate Event Records in Windows Event Viewer

Opened **Event Viewer → Windows Logs → Security** on DC10 and searched by Event Record ID to cross-reference the wazuh alerts.

**Event Record ID 17526** (from wazuh Rule ID 60112 alert):
- Event ID: **4719** — System audit policy was changed
- This is the last in a series of audit policy change events

**Event Record ID 17464** (from wazuh Rule ID 92653 — RDP logon):
- Confirms the RDP session to DC10 was initiated from **10.1.16.2 (MS10)**

| Field | Value |
|-------|-------|
| Logon type | **10 (RemoteInteractive)** |
| Source IP | 10.1.16.2 (MS10) |

> Scored question answered correctly: Logon type 10 = RemoteInteractive (Remote Desktop)

**Windows logon type reference:**

| Type | Title | Description |
|------|-------|-------------|
| 2 | Interactive | Direct keyboard logon |
| 3 | Network | Network resource access |
| 4 | Batch | Batch process logon |
| 7 | Unlock | Workstation unlock |
| 8 | NetworkCleartext | Network logon with plaintext credentials |
| 9 | NewCredentials | Outbound connection with alternate credentials |
| 10 | RemoteInteractive | RDP / Terminal Services |
| 11 | CachedInteractive | Cached domain credentials (no DC contact) |

---

## Investigation Summary

| Finding | Detail |
|---------|--------|
| Alert trigger | Wazuh Rule 60112 — 17 audit policy change events on DC10 |
| Username associated | jaime |
| Access method | RDP from MS10 (10.1.16.2) — not jaime's normal workstation (PC10) |
| Logon type | 10 (RemoteInteractive) |
| Action taken | All auditing categories set to No Auditing (Event ID 4719) |
| Significance | Attacker with jaime's credentials used MS10 to RDP into DC10 and disabled all auditing |

**Next step from the investigation:** The origin of the RDP session was MS10 — not PC10 (jaime's normal workstation). This raises the question: how did someone log into MS10 and gain access to the jaime credentials? The investigation continues in Lab 04.

---

## Key Takeaway

Root cause analysis requires following the evidence chain rather than stopping at the first finding. The wazuh alert identified the what (audit policy changes) and the who (jaime). Event Viewer on DC10 confirmed the how (RDP from an unexpected system). But the why and the full attack chain require expanding the investigation further — which is exactly how a real SOC investigation works.

---

*Write-up by Lereko Mohlomi | [LinkedIn](https://www.linkedin.com/in/lereko-mohlomi/) | [Back to Incident Response](./README.md)*
