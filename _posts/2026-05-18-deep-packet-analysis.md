---
title: "Deep Packet Analysis - DNS Tunneling Forensics"
date: 2026-05-18
categories: [Forensics, Network]
tags: [dns-tunneling, packet-capture, network-forensics, hex-decoding, base64]
author: yvngdrac
---

# Deep Packet Analysis

**Author:** yvngdrac  
**Date:** November 2025  
**Category:** Network Forensics  
**Difficulty:** Easy/Medium  
**Tools:** tshark, awk, xxd, base64

---

## Challenge Description

I got a packet capture file that needs deeper analysis beyond surface level. The challenge name "Deep Packet Inspection" was a pretty big hint that I need to look beyond it.

The file contained DNS traffic with **hidden data exfiltrated via DNS tunneling**.

---

## Solution

### Step 1: Download and Extract

First thing first, I needed to download the challenge file:

```bash
wget 'https://barqsec.exploit3rs.ae/files/7339bce8a8b6cf311b86864814378108/challenge.zip' -O challenge.zip
unzip challenge.zip
```

---

### Step 2: Initial Analysis

I started with the basics and analyzed the protocol hierarchy:

```bash
tshark -r challenge.pcap -q -z io,phs
```

**Output:**
```
===================================================================
Protocol Hierarchy Statistics
Filter:

eth                                      frames:376 bytes:35296
  arp                                    frames:20 bytes:840
  ip                                     frames:356 bytes:34456
    udp                                  frames:96 bytes:8492
      dns                                frames:96 bytes:8492
    tcp                                  frames:216 bytes:21620
      http                               frames:32 bytes:7812
    icmp                                 frames:44 bytes:4344
===================================================================
```

DNS traffic was present — a common spot for hiding data.

---

### Step 3: DNS Queries Analysis

Extracted DNS queries to look for anomalies:

```bash
tshark -r challenge.pcap -Y "dns" -T fields -e dns.qry.name | head -50
```

**Normal queries:**
- www.google.com
- api.github.com
- www.cloudflare.com
- stackoverflow.com
- www.reddit.com

**Suspicious queries:**
```
x091337ff-00-04-525868776247397064444e7963337445.telemetry.microsoft-cloud.com
x091337ff-01-04-4d7a4e77583141305932737a64463878.updates.cloudflare-cdn.com
x091337ff-02-04-626e4e774d324e304d54427558303030.telemetry.microsoft-cloud.com
x091337ff-03-04-6333517a636c38794d44493066513d3d.metrics.aws-cloudfront.net
```

**Pattern identified:** `x091337ff-[sequence]-04-[hex_data].[domain]`

The `x091337ff` prefix was clearly DNS tunneling for data exfiltration!

---

### Step 4: Extract and Decode Flag

Extracted the hex data from the suspicious DNS queries:

```bash
tshark -r challenge.pcap -Y "dns.qry.name contains \"x091337ff\"" -T fields -e dns.qry.name | sort -u | awk -F'-' '{print $4}' | awk -F'.' '{print $1}' | xxd -r -p | base64 -d
```

**Breaking down the command:**

1. `tshark` - extracts the suspicious DNS queries
2. `sort -u` - removes duplicates
3. `awk` - pulls out just the hex data part
4. `xxd -r -p` - converts hex to ASCII
5. `base64 -d` - decodes the final result

**Result:**
```
Exploit3rs{D33p_P4ck3t_1nsp3ct10n_M4st3r_2024}
```

---

## 🏆 The Flag

```
Exploit3rs{D33p_P4ck3t_1nsp3ct10n_M4st3r_2024}
```

---

## 📝 Key Takeaways

- **DNS can be abused** for data exfiltration via tunneling
- **Pattern recognition** is crucial in forensics (x091337ff prefix was the key)
- **Layered encoding** (hex → ASCII → base64) is common in CTFs
- **tshark filtering** makes it easy to isolate relevant traffic
- Always check for **out-of-place domain names** in packet captures

---

## 🔒 Detection & Prevention

- Monitor for **unusual DNS query patterns**
- Implement **DNS filtering** to block suspicious domains
- Use **DNS logging** to audit all queries
- Detect **encoded data in DNS queries** (hex, base64, etc.)
- Implement **network segmentation** to limit DNS exfiltration paths
