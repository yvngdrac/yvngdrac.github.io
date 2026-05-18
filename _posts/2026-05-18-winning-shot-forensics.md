---
title: "The Winning Shot — CTF Forensics Writeup"
date: 2026-05-18
categories: [CTF, Forensics]
tags: [core-dump, binary-analysis, memory-forensics, objdump, stack-layout]
author: yvngdrac
---

# The Winning Shot — CTF Forensics Writeup

**CTF:** UMCS CTF  
**Category:** Forensics  
**Flag:** `UMCS{k1n3t1c_3n3rgy_r3c0v3r3d_fr0m_c0r3_dump}`

---

## Overview

> "The break shot has been taken, and the balls are scattered across the memory space. We managed to freeze the table mid-game, but the suspect's final move remains hidden in the pocket."

This challenge provided a zip file (`8_b411_p00l_1s_7un.zip`) with a hint to use Linux. Three files inside:

1. `pool` — JPEG image (rabbit hole, but fits the billiards theme)
2. `winning_shot` — ELF 64-bit binary (not stripped)
3. `core.170390` — **core dump** of `winning_shot` mid-execution

The "frozen table mid-game" = the core dump. The process was caught mid-sleep.

---

## Recon — What's in the zip?

```bash
unzip 8_b411_p00l_1s_7un.zip
file pool winning_shot core.170390
```

**Output:**
```
pool:         JPEG image data, JFIF standard 1.01
winning_shot: ELF 64-bit LSB pie executable, x86-64, not stripped
core.170390:  ELF 64-bit LSB core file, x86-64, from './winning_shot'
```

---

## Step 1 — Strings on the Binary

First rule of binary analysis: run `strings`!

```bash
strings winning_shot | grep -E "UMCS|k1n|3t1c|dump"
```

**Output:**
```
UMCS{k1nH
3t1c_3n3H
rgy_r3c0H
v3r3d_frH
fr0m_c0rH
3_dump}
CUE_BALL_STATE_V1
```

The flag is **fragmented** — each chunk ends with `H` (garbage from byte boundaries). The `CUE_BALL_STATE_V1` marker is a custom struct that'll be important.

---

## Step 2 — Disassembly of `main`

```bash
objdump -d winning_shot | grep -A 200 "<main>"
```

**What the binary does:**

1. **`malloc(0x3c)`** — allocates 60 bytes (the "key buffer")
2. **Opens `/dev/urandom`** and reads 60 random bytes
3. **Loop (i = 0 to 14):** sets `key_buffer[i*4] = i+1` — each "ball" = `[ball_num, rand_byte, rand_byte, rand_byte]`
4. **Hardcoded flag** stored on stack via `movabs` instructions
5. **XOR loop:** `encrypted[i] = flag[i] XOR key[i % 60]`
6. Writes result to file `WINSHOT`
7. Prints PID and **sleeps forever** — that's why we have the core dump!

---

## Step 3 — Stack Layout Analysis

The `movabs` instructions in the disassembly:

```
mov %rax,-0x60(%rbp)   ; offset 0
mov %rdx,-0x58(%rbp)   ; offset 8
mov %rax,-0x50(%rbp)   ; offset 16
mov %rdx,-0x48(%rbp)   ; offset 24
mov %rax,-0x42(%rbp)   ; offset 30  ← NOT -0x40! Overlapping write
mov %rdx,-0x3a(%rbp)   ; offset 38
```

**Key finding:** The write at `-0x42` **overlaps** with the previous write. This packs the string tightly and is critical to decode correctly.

---

## Step 4 — Extracting the Key from Core Dump

Since the key is random (from `/dev/urandom`), it can't be reconstructed from the binary. But the process is frozen — the key is still in heap memory!

**Using pyelftools to parse the core dump:**

```python
from elftools.elf.elffile import ELFFile

with open('core.170390','rb') as f:
    elf = ELFFile(f)
    for seg in elf.iter_segments():
        print(seg.header.p_type, hex(seg.header.p_vaddr), hex(seg.header.p_filesz))
```

**Finding the marker in heap:**

```python
data = open('core.170390','rb').read()
heap = data[0x33f8:]  # heap segment offset
idx = heap.find(b'CUE_BALL_STATE_V1')
# Found at heap+752
```

**Dumping the key:**

```
offset 32: 01 f5 be 62   ← Ball 1:  rand=0xf5, 0xbe, 0x62
offset 36: 02 e4 70 5f   ← Ball 2:  rand=0xe4, 0x70, 0x5f
offset 40: 03 a5 1a 93   ← Ball 3:  ...
...
offset 88: 0f 29 e5 8a   ← Ball 15: rand=0x29, 0xe5, 0x8a
```

**All 15 balls (60 bytes) recovered!**

---

## Step 5 — Recovering the Flag

The flag is **already plaintext in the binary** — stored before XOR encryption. Just decode the stack layout correctly:

```python
import struct

flag_raw = bytearray(48)

def write_q(buf, offset, val):
    b = struct.pack('<Q', val)
    for i, x in enumerate(b):
        buf[offset+i] = x

write_q(flag_raw, 0,  0x6e316b7b53434d55)
write_q(flag_raw, 8,  0x336e335f63317433)
write_q(flag_raw, 16, 0x306333725f796772)
write_q(flag_raw, 24, 0x72665f6433723376)
write_q(flag_raw, 30, 0x7230635f6d307266)  # offset 30 = overlap!
write_q(flag_raw, 38, 0x7d706d75645f33)

flag = flag_raw.rstrip(b'\x00').decode()
print(flag)
```

**Output:**
```
UMCS{k1n3t1c_3n3rgy_r3c0v3r3d_fr0m_c0r3_dump}
```

---

## 🏆 The Flag

```
UMCS{k1n3t1c_3n3rgy_r3c0v3r3d_fr0m_c0r3_dump}
```

---

## Summary

| Step | What I did |
|---|---|
| Unzip + `file` | Identified binary, JPEG, and **core dump** |
| `strings` | Found fragmented flag chunks hinting at `movabs` storage |
| `objdump -d` | Reversed the XOR encryption logic and stack layout |
| Parse core dump | Located heap segment, found `CUE_BALL_STATE_V1` marker |
| Dump key | Extracted all 15 "ball" key entries (60 bytes) |
| Decode flag | Accounted for **overlapping `movabs` offsets** |

---

## Key Insights

- **Overlapping writes** in assembly can obscure data packing
- **Core dumps** preserve the entire process state, including heap memory
- **Custom markers** (like `CUE_BALL_STATE_V1`) are forensic goldmines
- **File operations** in the binary hint at output locations
- The pool/billiards theme was perfect: 15 balls = 15 key entries, "pocket" = encrypted output file, "core dump" = frozen table

---

## 📝 What I Learned

- How to parse ELF core dumps programmatically
- Understanding stack layout and memory overlaps
- Using `strings` + `objdump` for quick reconnaissance
- Importance of custom markers in forensic analysis
- XOR encryption can be meaningless if the plaintext is hardcoded elsewhere
