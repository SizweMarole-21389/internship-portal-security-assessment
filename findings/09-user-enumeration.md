[← Back to overview](../README.md)

# Finding 9: User Enumeration via Registration Responses

**Severity:** Low &nbsp;|&nbsp; **CVSS v3.1:** 5.3 (`AV:N/AC:L/PR:N/UI:N/S:U/C:L/I:N/A:N`)
**CWE:** CWE-204 - Observable Response Discrepancy
**OWASP Top 10 (2021):** A07:2021 Identification and Authentication Failures
**Proof captured:** No (documented, no screenshot evidence captured during testing)

## Description

The registration endpoint **POST /api/v1/auth/register** returns distinct error codes depending
on whether a value is already taken: **EMAIL_EXISTS** for a registered email and
**NICKNAME_EXISTS** for a taken nickname. An attacker can submit candidate values and use the
response to confirm which emails and nicknames are already registered, without needing valid
credentials for either.

## Reproduction Steps

1. Register with an email already in use, for example **admin@target.local**. The response is
   **EMAIL_EXISTS**.
2. Register with a taken nickname, for example **admin**. The response is **NICKNAME_EXISTS**.
3. A new, unused value returns a normal success, distinguishing registered from unregistered
   values.

## Business Impact

Enumeration confirms valid accounts and feeds the credential-stuffing and brute-force scenario
in Finding 8: an attacker can first build a confirmed list of real accounts, then focus
brute-force effort only on those. It also confirms which email addresses belong to real users,
which is useful for targeted phishing.

## Remediation

Return a single generic message for registration conflicts (e.g. "unable to complete
registration with the supplied details") rather than a field-specific code, and confirm account
details out of band - for example by email - rather than disclosing existence directly in the
response.

---

[← Finding 8](08-no-rate-limiting.md) &nbsp;|&nbsp; [Back to overview](../README.md) &nbsp;|&nbsp; [Next: Finding 10 - Missing HTTP Security Headers →](10-missing-security-headers.md)
