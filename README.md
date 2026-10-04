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





# Security Incident Report — Brute-Force Web Compromise

## Description

Cybersecurity incident analysis of a compromised website involving a brute-force attack, malicious file delivery, HTTP traffic, DNS redirection, and account-security remediation.

---

## Section 1: Identify the Network Protocol Involved in the Incident

The primary network protocol involved in the incident is **Hypertext Transfer Protocol (HTTP)**.

The issue involved users accessing the web server for `yummyrecipesforme.com`, and the tcpdump traffic log shows HTTP being used when the browser contacts the website. The malicious file was delivered to users over **HTTP**, which operates at the **application layer** of the TCP/IP model.

The traffic log also shows DNS requests before the HTTP connections. DNS was used to resolve the domain names `yummyrecipesforme.com` and later `greatrecipesforme.com` to IP addresses. However, the protocol directly associated with accessing the webpages and transferring the malicious content was **HTTP**.

Example traffic:

```text
HTTP: GET / HTTP/1.1
```

---

## Section 2: Document the Incident

Several customers contacted the website helpdesk after visiting `yummyrecipesforme.com`. They reported being prompted to download and run a file that claimed to provide access to free recipes. After running the file, their computers began operating more slowly and their browsers were redirected to a different website. The website owner also attempted to log in to the administrative panel but discovered that access to the account had been lost.

The cybersecurity analyst investigated the incident in a **sandbox environment** to avoid affecting the company network. The analyst opened `yummyrecipesforme.com`, ran **tcpdump** to capture the resulting network traffic, and reproduced the reported behavior. The website prompted the analyst to download an executable file presented as a browser update. After the file was downloaded and executed, the browser redirected to `greatrecipesforme.com`.

The tcpdump traffic log showed that the browser first sent a DNS request to resolve `yummyrecipesforme.com`. The DNS server returned the website's IP address, and the browser established a connection to the site using HTTP. After the suspicious file was downloaded and executed, the network traffic changed. The browser sent another DNS request, this time for `greatrecipesforme.com`, and then established an HTTP connection with that website. This change in traffic supported the observation that the downloaded file redirected users away from the legitimate website.

A senior cybersecurity analyst reviewed the website source code and the downloaded file. The investigation found that malicious JavaScript had been added to the website to prompt visitors to download the executable file. The downloaded file contained a script that redirected browsers to `greatrecipesforme.com`. The cybersecurity team determined that the web server had been compromised through a **brute-force attack** against the administrative account. The attack succeeded because the administrative password was still set to a known default password and there were no controls in place to limit repeated login attempts. After gaining access, the attacker changed the administrator password and modified the website source code.

---

## Section 3: Recommend One Remediation for Brute-Force Attacks

A recommended security measure is to implement **two-factor authentication (2FA)** for administrative accounts.

2FA requires a user to provide a password and an additional authentication factor, such as a one-time passcode. This means that even if an attacker successfully guesses or obtains an administrative password, the password alone would not be enough to gain access to the account.

Implementing 2FA would make a password-only brute-force attack much less likely to result in successful administrative access.

Additional supporting controls may include:

- Replacing all default passwords before systems are placed into production
- Limiting repeated login attempts
- Monitoring failed authentication attempts
- Preventing reuse of previous or default passwords
- Requiring strong administrative passwords

---

## Incident Evidence Summary

| Evidence | Observation |
|---|---|
| Legitimate domain | `yummyrecipesforme.com` |
| Redirected domain | `greatrecipesforme.com` |
| Primary web protocol | HTTP |
| Name-resolution protocol | DNS |
| Packet-analysis tool | tcpdump |
| Initial attack | Brute-force attack |
| Compromised account | Administrative account |
| Security weakness | Default password and lack of brute-force controls |
| Malicious modification | JavaScript added to website source |
| User impact | Malicious file download, browser redirection, slower computers |
| Recommended remediation | Two-factor authentication (2FA) |

---

## Traffic Sequence

```text
User visits yummyrecipesforme.com
        |
        v
DNS request for yummyrecipesforme.com
        |
        v
DNS response with legitimate IP address
        |
        v
HTTP connection to legitimate website
        |
        v
Malicious download prompt
        |
        v
Executable file downloaded and run
        |
        v
DNS request for greatrecipesforme.com
        |
        v
DNS response with redirected destination
        |
        v
HTTP connection to greatrecipesforme.com
```

---

## Key Findings

- HTTP was the primary protocol involved in accessing the website and transferring the malicious content.
- DNS was used to resolve both the legitimate and redirected domain names.
- The website source code had been modified with malicious JavaScript.
- Customers were prompted to download and execute a malicious file.
- The file redirected browsers from `yummyrecipesforme.com` to `greatrecipesforme.com`.
- The administrative account was compromised through a brute-force attack.
- The attack was successful because the administrative password was still set to a default value and repeated login attempts were not restricted.
- Two-factor authentication is recommended as a key remediation.

---

## Repository Purpose

This repository is part of a cybersecurity learning and professional portfolio. It demonstrates the ability to:

- Analyze tcpdump traffic
- Identify application-layer protocols
- Document a security incident
- Interpret DNS and HTTP traffic
- Trace malicious redirection behavior
- Identify a brute-force account compromise
- Recommend an authentication security control

---

## Disclaimer

This repository documents a simulated cybersecurity training scenario for educational and portfolio purposes.

---

## Suggested GitHub Topics

`cybersecurity` `tcpdump` `http` `dns` `brute-force` `incident-response` `web-security` `packet-analysis` `2fa`
