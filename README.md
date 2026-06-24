# Web Application Penetration Test — Internship Application Portal

A sanitised, portfolio write-up of a **black-box web application penetration test** against a
deliberately vulnerable internship-application portal, performed in a controlled lab. The
assessment started from the position of an ordinary registered user with no source-code or
backend access — the same starting point as any external attacker — and ended in full server
compromise and account takeover through a chain of four High-risk vulnerabilities.

> ⚠️ **Ethics & sanitisation.** This is an educational lab report. All client/organisation
> branding, hostnames, captured proof markers ("flags"), credentials, and password hashes have
> been removed or redacted, and the sensitive regions of every screenshot are blurred. Nothing
> here was performed against a live production system, and nothing here should be used to attack
> one. Testing was authorised and conducted in a lab environment.

---

## 📋 Findings at a Glance

| # | Finding | Severity | CVSS v3.1 | CWE | OWASP 2021 |
|---|---------|----------|-----------|-----|------------|
| 1 | Unrestricted File Upload → Remote Code Execution | **High** | 8.8 | CWE-434 | A03 / A05 |
| 2 | Error-Based SQL Injection (profile update) | **High** | 8.1 | CWE-89 | A03 |
| 3 | IDOR — Unauthorised Access to Any Profile | **High** | 6.5 | CWE-639 | A01 |
| 4 | Stored Cross-Site Scripting (surname field) | **High** | 6.1 | CWE-79 | A03 |
| 5 | Assignment Logic Flaws (scoring / retakes) | Low | 4.3 | CWE-840 | A04 |
| 6 | JWT Refresh Token Not Invalidated on Logout | Medium | 5.3 | CWE-613 | A07 |
| 7 | Weak Password Policy | Low | 3.7 | CWE-521 | A07 |
| 8 | No Rate Limiting on Authentication | Low | 5.3 | CWE-307 | A07 |
| 9 | Missing HTTP Security Headers | Low | 4.3 | CWE-693 / CWE-1021 | A05 |
| 10 | Information Disclosure (versions / verbose errors) | Info | 5.3 | CWE-200 / CWE-209 | A05 |

**Result:** 10 vulnerabilities — 4 High, 1 Medium, 4 Low, 1 Informational. Two High findings (file
upload and SQL injection) are each enough on their own to hand over the server and the full database.

➡️ **[Read the full report → REPORT.md](REPORT.md)**

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

## 🛠️ Tools & Techniques

- **Burp Suite Community Edition** — intercepting proxy, request replay, Intruder
- **curl** — direct crafted requests
- **Browser DevTools** — rendering / output-encoding verification
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

## 📁 Repository Structure

```
.
├── README.md            # this overview
├── REPORT.md            # full sanitised findings report (CWE / OWASP / CVSS, repro, remediation)
└── evidence/            # redacted screenshots, grouped per finding
    ├── Finding-01-RCE/
    ├── Finding-02-SQLi/
    ├── Finding-03-IDOR/
    ├── Finding-04-XSS/
    ├── Finding-05-Assignment/
    ├── Finding-06-JWT/
    ├── Finding-07-WeakPassword/
    ├── Finding-08-RateLimit/
    └── Finding-09-Headers/
```

## 📚 Standards & References

[OWASP Top 10 (2021)](https://owasp.org/Top10/) ·
[OWASP WSTG](https://owasp.org/www-project-web-security-testing-guide/) ·
[MITRE CWE](https://cwe.mitre.org/) ·
[CVSS v3.1](https://www.first.org/cvss/v3.1/specification-document) ·
[NIST NVD](https://nvd.nist.gov/)

---

*Authored by **Sizwe Marole**. Report version 2.0 (sanitised for public release).*
