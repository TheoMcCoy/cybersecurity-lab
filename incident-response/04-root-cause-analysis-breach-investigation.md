# Lab 04 — Root Cause Analysis: Full Breach Investigation

**Source:** CompTIA Security+ CertMaster Labs — Alerting and Monitoring Tools (Section 12.1.11)
**Environment:** MS10 → PC10 → ROUTER-BORDER (OPNsense firewall) → KALI
**Tools:** Wazuh SIEM, Windows Event Viewer, OPNsense firewall logs, Mozilla Thunderbird
**Technique:** Phishing → malicious batch script → browser proxy hijack → credential theft → RDP lateral movement → audit log tampering
**MITRE ATT&CK:** T1566.002 (Spearphishing Link) / T1204.002 (Malicious File) / T1090 (Proxy) / T1021.001 (RDP) / T1562.002 (Disable Audit Logging)
**Security+ Objectives:** 2.2 / 2.4 / 4.4 / 4.8 / 4.9
**Status:** ✅ Complete

---

## Scenario

Continuing from Lab 03, the RDP connection to DC10 originated from MS10. But MS10 is a server — the jaime account should not have been logged in there. This lab traces the full attack chain backwards to its origin, reconstructing the complete breach from initial access through to the anti-forensic activity observed in Labs 01–03.

> Simulated date: 31 March 2023.

---

## Environment

| System | Role | IP |
|--------|------|----|
| MS10 | Windows Server 2016 — RDP session origin | 10.1.16.2 |
| PC10 | Jaime's assigned workstation (Windows Server 2019) | 10.1.24.101 |
| ROUTER-BORDER | OPNsense network firewall | 10.1.128.253 |
| WAZUH | SIEM platform | 10.1.16.242 |
| KALI | Security workstation | — |
| juiceshop.com | External site — used as lure in phishing email | 203.0.113.228 |

---

## Part 1 — Investigate MS10: Who Was Actually Logged In?

### Step 1 — Identify the Real Logon Account on MS10

Connected to MS10 as administrator and opened **Event Viewer → Windows Logs → Security**.

Searched for events around 17:55 PM (just before the RDP session to DC10 at 17:55:26) with Account Name: jaime.

Located a **Logon event (Event ID 4624)** with Event Record ID **4548**:

- **Account used:** jaime (confirmed RDP session initiator)
- **Logon type:** 10 (RemoteInteractive)

Then searched for the **dylan** account and located Event Record ID **4178**:

- **Account:** dylan
- **Logon type:** 2 (Interactive — direct keyboard login at MS10)
- **Time:** 5:48 PM

> Scored question answered correctly: Logon type 2 (Interactive) for dylan on MS10.

**Finding:** Dylan (an HR employee with a standard, limited account) logged directly into MS10 at 5:48 PM. Shortly after, the jaime account was used from MS10 to RDP into DC10 and disable auditing. Dylan had obtained the jaime credentials and used them.

---

## Part 2 — Investigate PC10: How Did Dylan Get Jaime's Credentials?

### Step 2 — Review Jaime's Email on PC10

Connected to PC10 as jaime and opened **Mozilla Thunderbird**. Found a single email in the inbox: **"Grab you free juice!"**

The email subject used incorrect grammar — an immediate phishing indicator. Despite appearing to come from a known sender address, the content contained two suspicious elements:

1. A **System Update** link — hovering over it revealed a URL containing an IP address and a file named `proxyset.bat`
2. A link to **juiceshop.com** — appearing legitimate but suspicious in context

**Scam IP address:** `10.1.24.142`

---

### Step 3 — Confirm the Malicious File Is Present

From Command Prompt on PC10:

```cmd
cd c:\ && dir is proxyset.bat
```

**Result:** `proxyset.bat` confirmed present at `C:\Users\jaime\Downloads`

> Scored question answered correctly: `c:\Users\jaime\Downloads`

This confirms Jaime clicked the System Update link, downloaded the batch script, and executed it.

---

### Step 4 — Inspect the Batch Script's Effect

Viewed the contents of `proxyset.bat`:

```cmd
type c:\Users\jaime\Downloads\proxyset.bat
```

The script changed Firefox browser proxy settings — redirecting all Firefox traffic through **10.1.16.2 (MS10)** as a proxy server.

Confirmed in Firefox: **Settings → Network Settings** showed **Manual proxy configuration** pointing to MS10.

> Scored question answered correctly: Manual proxy configuration

**Effect:** Any website Jaime visited through Firefox after executing this script had its traffic routed through MS10 — where Dylan was logged in and monitoring.

![Phishing email and proxyset.bat contents on PC10](../screenshots/proxyset-bat-email.jpg)
*PC10 — Jaime's inbox showing the "Grab you free juice!" phishing email. The CMD window overlaid shows the full contents of proxyset.bat: a script that rewrites Firefox's prefs.js to route all HTTP, SOCKS, and SSL traffic through 10.1.16.2 (MS10) on port 8080.*

---

## Part 3 — Investigate ROUTER-BORDER: Confirm the Proxy Chain

### Step 5 — Verify the Juice Shop Connection via Firewall Logs

Accessed the OPNsense firewall management interface at `https://10.1.128.253`.

Navigated to **Firewall → Log Files → Live View**.

**DNS resolution for juiceshop.com:**

```bash
ping juiceshop.com -c 1
# Result: 203.0.113.228
```

Filtered firewall logs:
- `dst` = `203.0.113.228` (juiceshop.com) — confirmed a connection occurred
- `src` = `10.1.16.2` (MS10) — the traffic came **from MS10**, not directly from PC10

This confirmed that Jaime's browser, configured to use MS10 as a proxy, routed the Juice Shop connection through MS10.

**Destination port:** `80` — plaintext HTTP (not HTTPS)

> Scored question answered correctly: Port 80 — plaintext communication

**Timestamp on the firewall event:** `2023-04-01T00:54:25` UTC = `5:54 PM` local (Pacific Time, UTC-7h) on 31 March 2023 — consistent with the rest of the attack timeline.

---

## Reconstructed Attack Timeline

| Time (Local) | Event | System |
|-------------|-------|--------|
| Unknown | Dylan sends phishing email to Jaime with `proxyset.bat` link | External |
| Unknown | Jaime receives email, clicks System Update link, downloads `proxyset.bat` | PC10 |
| Unknown | Jaime executes `proxyset.bat` — Firefox proxy set to MS10 (10.1.16.2) | PC10 |
| 5:32 PM | Dylan logs into MS10 directly (Interactive logon, type 2) | MS10 |
| ~5:54 PM | Jaime visits Juice Shop link from email — traffic routed through MS10 | PC10 → MS10 |
| ~5:54 PM | Dylan intercepts Jaime's HTTP traffic on MS10 — captures jaime credentials | MS10 |
| 5:55 PM | Dylan uses jaime credentials to RDP from MS10 to DC10 (logon type 10) | MS10 → DC10 |
| 5:56 PM | Dylan (as jaime) disables all audit policies on DC10 (Event ID 4719, ×17) | DC10 |

---

## Full Attack Chain

```
Phishing email (proxyset.bat)
        ↓
Jaime clicks link → downloads + executes batch script
        ↓
Firefox proxy hijacked → all traffic routed through MS10
        ↓
Jaime visits juiceshop.com → plaintext HTTP credentials intercepted by Dylan on MS10
        ↓
Dylan uses jaime credentials → RDP from MS10 to DC10
        ↓
Dylan (as jaime) disables all auditing on DC10
        ↓
Evidence destroyed — but wazuh caught it all
```

---

## Key Takeaway

This investigation demonstrates the full lifecycle of an insider-facilitated attack using social engineering, proxy hijacking, credential theft, and anti-forensic activity — all within a single organisation. Several controls would have broken this chain at different points: HTTPS enforcement (prevents credential interception), email filtering (blocks phishing), browser policy lockdown (prevents proxy changes), and network segmentation (prevents a standard HR employee from logging directly into a production server). The fact that wazuh caught the audit policy changes — even after Dylan tried to cover his tracks — validates the value of a SIEM that receives events before they can be deleted at the source.

---

*Write-up by Lereko Mohlomi | [LinkedIn](https://www.linkedin.com/in/lereko-mohlomi/) | [Back to Incident Response](./README.md)*
