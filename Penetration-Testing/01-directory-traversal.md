# Lab: Directory Traversal

**Source:** CompTIA Security+ CertMaster Labs (Section 15.3.4 — Performing Reconnaissance)
**Environment:** DVWA (Damn Vulnerable Web Application) on Kali Linux
**Technique:** Path Traversal / Local File Inclusion (LFI)
**OWASP Category:** A01 — Broken Access Control
**Status:** ✅ Complete

---

## Scenario

A web application running on a simulated enterprise server was suspected of being vulnerable to directory traversal. The objective was to confirm the vulnerability, determine the extent of file system access achievable through URL manipulation, and document the attack path for defensive remediation.

---

## What is Directory Traversal?

Directory traversal (also called path traversal) is a vulnerability that allows an attacker to read files outside the web root directory by manipulating URL parameters. When a web application passes user-supplied input directly into file system calls without sanitisation, an attacker can use `../` sequences to navigate up the directory tree and access sensitive system files.

---

## Methodology

### Step 1 — Confirming the Vulnerability

Navigated to the File Inclusion page on DVWA and observed the URL structure:

```
dvwa.structureality.com/vulnerabilities/fi/?page=include.php
```

The `page=` parameter accepts a filename and includes it directly — a classic LFI pattern.

**Test 1 — Non-existent directory:**
```
?page=blah/include.php
```
Result: Blank page with error statements — confirms the parameter is being processed by the file system.

**Test 2 — Parent directory traversal:**
```
?page=blah/../include.php
```
Result: Original File Inclusion page displayed — confirms `../` traversal is supported.

---

### Step 2 — Accessing Sensitive Files

**Attempt 1 — Relative reference without leading slash:**
```
?page=etc/passwd
```
Result: Blank — application requires correct path resolution.

**Attempt 2 — Absolute reference:**
```
?page=/etc/passwd
```
Result: Contents of `/etc/passwd` displayed — confirms absolute path traversal works.

**Attempt 3 — Relative traversal to `/etc/passwd`:**
```
?page=../../etc/passwd
```
Result: Blank — web root not at expected depth.

After adding additional `../` sequences:
```
?page=../../../etc/passwd
```
Result: `/etc/passwd` contents displayed — relative traversal confirmed at correct depth.

---

### Step 3 — OS Version Enumeration

```
?page=../../proc/version
```
Result: Linux kernel version and OS details retrieved from `/proc/version`.

**Additional `/proc` files explored:**
- `/proc/cpuinfo` — CPU architecture details
- `/proc/meminfo` — Memory configuration
- `/proc/uptime` — System uptime

**Attempt to access `/etc/shadow`:**
```
?page=/etc/shadow
```
Result: Permission denied — shadow file restricted to root. This is expected and confirms file access is limited to the web user context, not root.

---

## Attack Summary

| Technique | URL Pattern | Result |
|-----------|------------|--------|
| Absolute path | `?page=/etc/passwd` | ✅ File contents exposed |
| Relative traversal | `?page=../../../etc/passwd` | ✅ File contents exposed |
| OS enumeration | `?page=../../proc/version` | ✅ Kernel version exposed |
| Privileged file access | `?page=/etc/shadow` | ❌ Permission denied (expected) |

---

## Impact Assessment

An attacker exploiting this vulnerability could read:
- `/etc/passwd` — usernames, home directories, shell types
- `/proc/version` — OS and kernel version (useful for selecting kernel exploits)
- Application config files potentially containing database credentials
- Any file readable by the web server user context

---

## Defensive Recommendations

| Control | Description |
|---------|-------------|
| Input validation | Reject any input containing `../`, `./`, or absolute path characters |
| Allowlist approach | Define an explicit list of permitted filenames — reject everything else |
| Chroot / containerisation | Restrict the web application's file system view to the web root only |
| Least privilege | Run the web server as a low-privilege user with minimal file read permissions |
| Web Application Firewall | Deploy WAF rules to detect and block traversal patterns in URL parameters |

---

## Key Takeaway

The core issue is **unsanitised user input passed directly to file system operations**. This is one of the most fundamental web application security failures and remains prevalent because developers often trust that URL parameters will only contain expected values. The fix is always to validate and sanitise input before it touches the file system — never trust user-supplied path components.

---

*Write-up by Lereko Mohlomi | [LinkedIn](https://www.linkedin.com/in/lereko-mohlomi/)*
