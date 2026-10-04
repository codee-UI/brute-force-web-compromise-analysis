# Brute-Force Web Compromise & Malicious Redirect Analysis

## Repository Description

Security incident analysis of a compromised web server involving a brute-force attack, malicious JavaScript injection, malware delivery, DNS resolution, HTTP traffic, and browser redirection.

## Overview

This repository documents a simulated cybersecurity incident involving `yummyrecipesforme.com`.

A former employee gained unauthorized access to the website's administrative account by repeatedly attempting known default passwords. After gaining access, the attacker modified the website source code, inserted malicious JavaScript, prompted visitors to download an executable file, changed the administrative password, and redirected affected users to `greatrecipesforme.com`.

The incident was investigated in a sandbox using **tcpdump**.

## Incident Summary

- **Legitimate website:** `yummyrecipesforme.com`
- **Malicious redirect:** `greatrecipesforme.com`
- **Initial attack:** Brute-force attack
- **Compromised account:** Administrative account
- **Primary weakness:** Default password remained enabled
- **Network analyzer:** tcpdump
- **Protocols observed:** DNS, TCP, HTTP
- **Primary recommendation:** Multi-factor authentication (MFA)

---

# Section 1 — Network Protocols Identified

## DNS

DNS was used to resolve domain names into IP addresses.

The first DNS request was for:

`yummyrecipesforme.com`

The DNS server returned:

`203.0.113.22`

Example:

```text
your.machine.52444 > dns.google.domain
A? yummyrecipesforme.com

dns.google.domain > your.machine.52444
A 203.0.113.22
```

Later, the browser generated another DNS request for:

`greatrecipesforme.com`

after the malicious executable redirected the browser.

## TCP

TCP was used to establish reliable connections between the client and the web servers.

The tcpdump log includes:

```text
[S]   SYN
[S.]  SYN-ACK
[.]   ACK
```

Example:

```text
your.machine.36086 > yummyrecipesforme.com.http
Flags [S]

yummyrecipesforme.com.http > your.machine.36086
Flags [S.]

your.machine.36086 > yummyrecipesforme.com.http
Flags [.]
```

## HTTP

HTTP was used to request website content over port 80.

The packet capture shows:

```text
HTTP: GET / HTTP/1.1
```

The supporting material indicates that this request may correspond to the webpage request or malicious file download.

## Protocol Summary

| Protocol | Role |
|---|---|
| DNS | Resolves domain names to IP addresses |
| TCP | Establishes reliable client-server connections |
| HTTP | Transfers webpage requests and content |
| IP | Routes packets between systems |

---

# Section 2 — Incident Documentation

## How the Incident Was Discovered

Customers reported that:

- The website prompted them to download a file to access free recipes.
- Their browser address changed after running the file.
- Their computers began running more slowly.

The website owner also discovered that they could no longer access the administrative panel.

The issue was escalated to the hosting provider and cybersecurity team.

## Investigation

The cybersecurity team created a sandbox and:

1. Started tcpdump.
2. Visited `yummyrecipesforme.com`.
3. Observed an executable download prompt.
4. Ran the file in the sandbox.
5. Observed redirection to `greatrecipesforme.com`.
6. Reviewed DNS, TCP, and HTTP traffic.
7. Examined the website source code.
8. Analyzed the downloaded file.

## Traffic Sequence

### 1. DNS lookup for the legitimate website

```text
14:18:32.192571
your.machine.52444 > dns.google.domain
A? yummyrecipesforme.com

14:18:32.204388
dns.google.domain > your.machine.52444
A 203.0.113.22
```

### 2. TCP connection to the legitimate website

```text
your.machine.36086 > yummyrecipesforme.com.http
Flags [S]

yummyrecipesforme.com.http > your.machine.36086
Flags [S.]

your.machine.36086 > yummyrecipesforme.com.http
Flags [.]
```

### 3. HTTP request

```text
HTTP: GET / HTTP/1.1
```

### 4. Malicious executable download

The compromised site contained JavaScript that prompted visitors to download and run an executable file.

### 5. DNS lookup for the malicious website

The browser then requested DNS resolution for:

`greatrecipesforme.com`

The raw tcpdump log shows the returned IP as:

`192.0.2.17`

> Note: the explanatory reading lists `192.0.2.172`, while the raw tcpdump log lists `192.0.2.17`. For exact packet evidence, the raw tcpdump capture should be treated as the primary source.

### 6. TCP and HTTP connection to the malicious website

```text
your.machine.56378 > greatrecipesforme.com.http
Flags [S]

greatrecipesforme.com.http > your.machine.56378
Flags [S.]

your.machine.56378 > greatrecipesforme.com.http
Flags [.]
```

The browser then issued:

```text
HTTP: GET / HTTP/1.1
```

---

# Incident Flow

```text
User visits yummyrecipesforme.com
            |
            v
        DNS lookup
            |
            v
        203.0.113.22
            |
            v
       TCP connection
            |
            v
        HTTP request
            |
            v
  Compromised webpage
            |
            v
 Malicious JavaScript prompt
            |
            v
 Executable downloaded and run
            |
            v
 DNS lookup for greatrecipesforme.com
            |
            v
      TCP connection
            |
            v
        HTTP request
            |
            v
 Redirect to malicious website
```

---

# Root Cause

The web server was compromised through a **brute-force attack** against the administrative account.

The attack succeeded because:

- The administrative password was still set to a default password.
- The password was easy to guess.
- Repeated login attempts were not restricted.
- No effective brute-force protection was enabled.

After obtaining administrative access, the attacker modified the website source code and changed the admin password.

---

# Attack Chain

```text
Default admin password
        |
        v
Repeated password attempts
        |
        v
Brute-force success
        |
        v
Administrative access
        |
        v
Website source modified
        |
        v
Malicious JavaScript added
        |
        v
Executable download prompt
        |
        v
Browser redirected
        |
        v
Malicious website reached
```

---

# Evidence

The findings are supported by:

- Customer helpdesk reports
- Website owner's failed admin login
- tcpdump packet capture
- DNS lookup records
- TCP connection records
- HTTP GET requests
- Website source-code review
- Analysis of the executable
- Senior analyst confirmation of compromise

---

# Security Impact

## Confidentiality

Potentially affected because users were redirected to a malicious website and malware was executed on their systems.

## Integrity

**Directly affected** because the attacker modified the legitimate website's source code.

## Availability

Potentially affected because the legitimate administrator lost access after the attacker changed the password.

---

# Section 3 — Recommended Remediation

## Require Multi-Factor Authentication (MFA)

The recommended security measure is to require **multi-factor authentication (MFA)** for administrative accounts.

MFA adds an authentication factor beyond the password.

Example:

```text
Password
   +
Authenticator code
   =
Administrative access
```

Even if an attacker guesses or obtains the password, the attacker would still need the second authentication factor.

This makes password-only brute-force attacks significantly less likely to result in a successful administrative account compromise.

---

# Additional Defensive Measures

A defense-in-depth approach can also include:

- Remove default passwords before deployment
- Require strong passwords
- Limit repeated login attempts
- Use account lockout or progressive delays
- Monitor failed authentication attempts
- Alert on suspicious administrative logins
- Restrict administrative access
- Apply least privilege
- Review website source-code integrity
- Maintain secure backups
- Monitor web applications

---

# Key Findings

1. A former employee targeted the administrative account.
2. Repeated default-password attempts were used.
3. The admin account still used a default password.
4. The brute-force attack succeeded.
5. The attacker modified the website source code.
6. Malicious JavaScript prompted users to download an executable.
7. The executable redirected browsers to `greatrecipesforme.com`.
8. DNS resolved both the legitimate and malicious domains.
9. TCP established the connections.
10. HTTP carried the webpage requests.
11. Customers reported redirection and slower computers.
12. MFA is recommended to reduce password-only account compromise.

---

# Skills Demonstrated

- Cybersecurity incident analysis
- tcpdump
- Packet analysis
- DNS
- TCP
- HTTP
- TCP/IP networking
- Brute-force attack analysis
- Web server compromise analysis
- Malicious JavaScript identification
- Malware delivery analysis
- Browser redirection analysis
- Incident documentation
- Authentication security
- MFA
- Defense in depth

---

# Repository Purpose

This repository is maintained as part of a cybersecurity learning and professional portfolio.

It demonstrates the ability to:

- Analyze packet-capture evidence
- Identify protocols involved in a security incident
- Reconstruct an attack sequence
- Document incident findings
- Identify the root cause of compromise
- Recommend an appropriate security control

---

# Disclaimer

This repository documents a simulated cybersecurity training scenario and is intended for educational and portfolio purposes.

The domains, IP addresses, users, organization, and security incident described here are part of the training environment.

---

## Suggested GitHub Topics

`cybersecurity` `tcpdump` `network-security` `dns` `tcp` `http` `brute-force` `malware-analysis` `incident-response` `web-security` `mfa` `packet-analysis`

## Keywords

`Cybersecurity` `Network Security` `tcpdump` `DNS` `TCP` `HTTP` `Brute Force Attack` `Web Server Compromise` `Malware` `JavaScript Injection` `Incident Response` `Packet Analysis` `MFA` `Authentication Security`
