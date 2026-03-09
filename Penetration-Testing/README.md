# Penetration Testing — Web Application Security

**Source:** CompTIA Security+ CertMaster Labs (Section 15.3.5 — Performing Penetration Testing)
**Environment:** DVWA (Damn Vulnerable Web Application) on Kali Linux
**Status:** ✅ Complete

---

## Overview

This folder contains write-ups from hands-on penetration testing labs conducted in a controlled environment. All labs were performed against DVWA (Damn Vulnerable Web Application) running on Kali Linux as part of the CompTIA Security+ CertMaster Lab pathway.

The three labs form a connected attack chain — each technique building on the previous, ultimately achieving an interactive OS-level reverse shell session starting from a single vulnerable web application.

---

## Attack Chain

```
Directory Traversal
        ↓
Command Injection
        ↓
File Upload → PHP Web Shell
        ↓
Reverse Shell (msfvenom + Metasploit)
        ↓
Interactive OS-Level Access (Meterpreter)
```

---

## Labs in This Folder

| # | Lab | Technique | OWASP Category |
|---|-----|-----------|---------------|
| 01 | [Directory Traversal](./01-directory-traversal.md) | Path Traversal / LFI | A01 Broken Access Control |
| 02 | [Command Injection](./02-command-injection.md) | OS Command Injection | A03 Injection |
| 03 | [File Upload & Reverse Shell](./03-file-upload-webshell.md) | RCE via File Upload + Metasploit | A04 Insecure Design |

---

## Tools Used

| Tool | Purpose |
|------|---------|
| Kali Linux | Attacker machine and lab environment |
| DVWA | Deliberately vulnerable web application target |
| Metasploit Framework | Exploitation framework and reverse shell handler |
| msfvenom | Malicious payload generation |
| Vim | Payload and web shell authoring |
| Firefox | Web application interaction and URL manipulation |

---

## Key Takeaway

These labs demonstrate how a single misconfigured web application can be compromised through a chain of escalating techniques — from reading sensitive files via URL manipulation, to injecting OS commands through a web form, to establishing a full interactive reverse shell using a malicious file upload. No single vulnerability existed in isolation; the real risk came from the combination.

---

*Write-ups by Lereko Mohlomi | [LinkedIn](https://www.linkedin.com/in/lereko-mohlomi/) | [Back to Main Repo](../../README.md)*
