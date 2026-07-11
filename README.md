# Web Application Penetration Test - Web Application Portal

A sanitised, portfolio write-up of a **black-box web application penetration test** against a
deliberately vulnerable web application portal, performed in a controlled lab. The
assessment started from the position of an ordinary registered user with no source-code or
backend access - the same starting point as any external attacker - and ended in full server
compromise and account takeover through a chain of four High-risk vulnerabilities.

> ⚠️ **Ethics & sanitisation.** This is an educational lab report. All client/organisation
> branding, hostnames, captured proof markers ("flags"), credentials, and password hashes have
> been removed or redacted, and the sensitive regions of every screenshot are blurred. Nothing
> here was performed against a live production system, and nothing here should be used to attack
> one. Testing was authorised and conducted in a lab environment.

**Assessor:** Sizwe Marole &nbsp;|&nbsp; **Assessment type:** Black-box web application penetration test
**Target (anonymised):** `portal.target.local` &nbsp;|&nbsp; **Report version:** 2.1 (sanitised for public release)

---

## Executive Summary

A black-box penetration test was conducted against an web application portal
from the position of a registered user with no access to source code or backend
configuration - the same starting point available to any external attacker. The
entire web application was in scope; the underlying hosting infrastructure was out of
scope. Eleven vulnerabilities were identified: **four High, one Medium, four Low, and two
Informational.**

Four of the eleven are High. The worst is an unrestricted file upload: a normal user
account can run commands on the server and take it over completely with a single
request. A second, in the profile-update endpoint, leaks the whole database on one
authenticated request, including the email address and password hash of every user.
The other two let any user read any other user's private profile, and store
JavaScript that executes in another user's browser, including the administrator's.

Each issue is serious on its own. Chained together they are worse. The database leak
exposes the file references for every candidate's CV, and the CV directory requires no
authentication, so those references are all an attacker needs to download the files.
The stored JavaScript can capture a live session token; if an administrator views the
malicious profile, the attacker inherits the admin session. Logging out does not help,
because the refresh token remains valid and a new session can be minted from it.

The end result is full account takeover and a level of access a normal user should
never have. As it stands the portal is critically vulnerable and should not go live
until the High-risk issues are fixed - in order: file upload, SQL injection, then the
access-control and cross-site-scripting issues.

---

## 📋 Findings at a Glance

| # | Finding | Severity | CVSS v3.1 | CWE | OWASP 2021 |
|---|---------|----------|-----------|-----|------------|
| 1 | [Unrestricted File Upload → Remote Code Execution](findings/01-unrestricted-file-upload.md) | **High** | 8.8 | CWE-434 | A03 / A05 |
| 2 | [Error-Based SQL Injection (profile update)](findings/02-sql-injection.md) | **High** | 8.1 | CWE-89 | A03 |
| 3 | [IDOR - Unauthorised Access to Any Profile](findings/03-idor.md) | **High** | 6.5 | CWE-639 | A01 |
| 4 | [Stored Cross-Site Scripting (surname field)](findings/04-stored-xss.md) | **High** | 6.1 | CWE-79 | A03 |
| 5 | [Assignment Logic Flaws (scoring / retakes)](findings/05-assignment-logic-flaws.md) | Low | 4.3 | CWE-840 | A04 |
| 6 | [JWT Refresh Token Not Invalidated on Logout](findings/06-jwt-refresh-token.md) | Medium | 5.3 | CWE-613 | A07 |
| 7 | [Weak Password Policy](findings/07-weak-password-policy.md) | Low | 3.7 | CWE-521 | A07 |
| 8 | [No Rate Limiting on Authentication](findings/08-no-rate-limiting.md) | Low | 5.3 | CWE-307 | A07 |
| 9 | [User Enumeration via Registration Responses](findings/09-user-enumeration.md) | Low | 5.3 | CWE-204 | A07 |
| 10 | [Missing HTTP Security Headers](findings/10-missing-security-headers.md) | Low | 4.3 | CWE-693 / CWE-1021 | A05 |
| 11 | [Information Disclosure (versions / verbose errors)](findings/11-information-disclosure.md) | Info | 5.3 | CWE-200 / CWE-209 | A05 |

**Result:** 11 vulnerabilities - 4 High, 1 Medium, 4 Low, 2 Informational. Two High findings (file
upload and SQL injection) are each enough on their own to hand over the server and the full database.

Click any finding above for its full write-up: description, reproduction steps, screenshots with
inline captions, business impact, and remediation - all self-contained in one file.

> **CVSS vs engagement severity.** A few findings carry a higher *engagement* severity
> than their raw CVSS base score because the business impact (volume of personal data,
> position in the attack chain) elevates real-world risk above the technical base
> metric. Both are shown so the reader can see the reasoning.

---

## 🧩 The Attack Chain

The High findings are dangerous individually, but they are designed to feed one another:

```
Unrestricted Upload (F1) ──► RCE as www-data ──► read DB creds, source, every CV
        │
SQL Injection (F2) ──► dump users + bcrypt hashes + CV UUIDs
        │
        └──► CV UUIDs + unauthenticated /cv-view (F1) ──► download every candidate's CV
IDOR (F3) ──► read any profile (incl. admin) without authorisation
Stored XSS (F4) ──► steal a live admin session token from browser storage
        │
JWT refresh flaw (F6) ──► session survives logout ──► persistent admin access
```

One root cause runs through all four High findings: **user input reaches sensitive operations with
no validation or output encoding.**

---

## Methodology

All testing was manual. Burp Suite Community Edition intercepted and replayed requests,
`curl` sent crafted requests directly, and browser developer tools showed how responses
rendered. No automated scanners were run; the SQL injection was extracted by hand with
error-based `extractvalue()` payloads rather than a tool such as sqlmap.

The application was first walked through with a normal user account, mapping every
page, form, and API endpoint while Burp recorded the traffic. The file upload, the input
fields, the access-control checks, and the login flow were marked as priority test areas.
Each endpoint was then tested individually: a clean request first to observe normal
behaviour, then test payloads to watch for errors, unexpected output, or inconsistency.
Once a weakness was confirmed it was written up with the exact request, the response, and
reproduction steps, and screenshots were captured along the way.

This methodology aligns with the **OWASP Web Security Testing Guide (WSTG)** and findings
are classified using **CWE**, mapped to the **OWASP Top 10 (2021)**, and scored with
**CVSS v3.1**.

## Risk Rating Definitions

| Rating | Definition |
| --- | --- |
| **High** | Potential for an attacker to control, alter, or delete electronic assets. Unauthorised access, data capture, defacement. |
| **Medium** | Combined with other factors, potential for asset compromise. Conditional or configuration-dependent exploitation. |
| **Low** | Likelihood or impact of exploitation is extremely low. Weak ciphers, outdated protocols, programmatic CAPTCHA bypass. |
| **Informational** | Cannot be exploited directly but not aligned with best practice. Information disclosure, version leakage. |

---

## 🛠️ Tools & Techniques

- **Burp Suite Community Edition** - intercepting proxy, request replay, Intruder
- **curl** - direct crafted requests
- **Browser DevTools** - rendering / output-encoding verification
- Manual, hand-built **error-based SQL injection** with MariaDB `extractvalue()` (no sqlmap)
- WAF blocklist bypass for stored XSS (`<h4 tabindex onfocus>`)
- JWT decoding and session-handling analysis
- Methodology aligned to the **OWASP Web Security Testing Guide (WSTG)**

## 🎯 Skills Demonstrated

- Black-box web application penetration testing end to end
- Vulnerability classification with **CWE**, **OWASP Top 10 (2021)**, and **CVSS v3.1** scoring
- Chaining individual issues into a realistic, high-impact attack path
- Clear, business-focused reporting with reproduction steps and remediation
- Mapping technical findings to legal/regulatory impact (**POPIA**, Sections 19 & 22)

---

## Conclusion

A self-registered account is enough to take full control of the server and keep that access after
logging out. The portal should not go live until the High-risk findings are fixed. The legal risk is
real, not theoretical: the Information Regulator's first POPIA fine - R5 million against a government
department in 2023 - came from this same pattern of a security breach and a failure of the duties in
Sections 19 and 22.

Eleven vulnerabilities were found: four High, one Medium, four Low, and two Informational. Two of the
High findings - the file upload and the SQL injection - are each enough on their own to hand over the
server and the full database. The bigger problem is how the High findings feed each other (Findings 1
to 4): what one leaks becomes the way into the next, ending in an admin takeover and access that
survives logout.

Fix in order of risk: the file upload first, then the SQL injection, then the access-control and
cross-site-scripting issues. The same root cause runs through all four High findings - user input
reaches sensitive operations with no validation or output encoding. One validation and output-encoding
standard, applied to every endpoint, fixes the whole class of bug instead of these few cases. The
authentication, password, enumeration, header, and information-disclosure findings can be handled once
the High-risk work is done.

---

## 📁 Repository Structure

```
.
├── README.md                        # this file - overview, methodology, findings table
├── findings/                        # one self-contained write-up per finding
│   ├── 01-unrestricted-file-upload.md
│   ├── 02-sql-injection.md
│   ├── 03-idor.md
│   ├── 04-stored-xss.md
│   ├── 05-assignment-logic-flaws.md
│   ├── 06-jwt-refresh-token.md
│   ├── 07-weak-password-policy.md
│   ├── 08-no-rate-limiting.md
│   ├── 09-user-enumeration.md
│   ├── 10-missing-security-headers.md
│   └── 11-information-disclosure.md
└── evidence/                        # redacted screenshots, referenced inline by findings/*.md
    ├── Finding-01-RCE/
    ├── Finding-02-SQLi/
    ├── Finding-03-IDOR/
    ├── Finding-04-XSS/
    ├── Finding-05-Assignment/
    ├── Finding-06-JWT/
    ├── Finding-07-WeakPassword/
    ├── Finding-08-RateLimit/
    └── Finding-10-Headers/
```

## 📚 Standards & References

- **OWASP Top 10 (2021)** - https://owasp.org/Top10/
- **OWASP Web Security Testing Guide (WSTG)** - https://owasp.org/www-project-web-security-testing-guide/
- **MITRE CWE** - https://cwe.mitre.org/ (CWE-434, 89, 639, 79, 840, 613, 521, 307, 204, 693, 1021, 200, 209)
- **CVSS v3.1 Specification** - https://www.first.org/cvss/v3.1/specification-document
- **NIST National Vulnerability Database (CVE lookup)** - https://nvd.nist.gov/
- **POPIA** - Protection of Personal Information Act 4 of 2013 (South Africa), Sections 19 & 22

---

*Authored by **Sizwe Marole**. Report version 2.1 (sanitised for public release).*
