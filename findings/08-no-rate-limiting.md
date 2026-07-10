[← Back to overview](../README.md)

# Finding 8: No Rate Limiting on Authentication

**Severity:** Low &nbsp;|&nbsp; **CVSS v3.1:** 5.3 (`AV:N/AC:L/PR:N/UI:N/S:U/C:L/I:N/A:N`)
**CWE:** CWE-307 - Improper Restriction of Excessive Authentication Attempts
**OWASP Top 10 (2021):** A07:2021 Identification and Authentication Failures
**Proof captured:** N/A

## Description

The login endpoint **POST /api/v1/auth/login** applies no rate limiting, lockout,
or throttling. Repeated failed attempts are all processed and return **401** with no **429**, no
delay, and no account lockout, permitting unrestricted brute-force and credential-stuffing attacks.

## Reproduction Steps

1. Send many login requests in quick succession with an incorrect password, using Burp Intruder or a script.
2. Every request returns 401 with no throttling, no 429, and no increasing delay.

   ![30 attempts all return 401 with no 429 or lockout](../evidence/Finding-08-RateLimit/SS-F08-01_ratelimit-ffuf-bruteforce.png)
   *__Figure 8.1__ - 30 attempts all return 401 with no 429 or lockout.*

## Business Impact

Combined with the leaderboard exposing 120+ valid nicknames and the weak
password policy (Finding 7), an attacker can run large credential-stuffing or brute-force campaigns
against confirmed accounts with no server-side restriction.

## Remediation

Apply per-account rate limiting as the primary control (per-IP limits are easily
evaded by rotating addresses), and add a CAPTCHA or step-up challenge after repeated failures.
Return **429** with a retry-after header and apply temporary lockout or increasing backoff.

---

[← Finding 7](07-weak-password-policy.md) &nbsp;|&nbsp; [Back to overview](../README.md) &nbsp;|&nbsp; [Next: Finding 9 - User Enumeration →](09-user-enumeration.md)
