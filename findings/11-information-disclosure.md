[← Back to overview](../README.md)

# Finding 11: Information Disclosure - Server Version Headers and Verbose Database Errors

**Severity:** Informational &nbsp;|&nbsp; **CVSS v3.1:** 5.3 (`AV:N/AC:L/PR:N/UI:N/S:U/C:L/I:N/A:N`)
**CWE:** CWE-200 - Exposure of Sensitive Information · CWE-209 - Generation of Error Message Containing Sensitive Information
**OWASP Top 10 (2021):** A05:2021 Security Misconfiguration
**Proof captured:** N/A

## Description

The application discloses exact software versions through several channels. The
**Server** header reads **nginx/1.24.0 (Ubuntu)** on all responses, and responses served by the PHP
backend include **X-Powered-By: PHP/8.x**. In addition, the profile update endpoint returns raw
MariaDB error messages on a malformed query, confirming the engine and version (**MariaDB 10.11.14**).
The verbose errors are the same behaviour that enables the SQL injection in Finding 2.

## Reproduction Steps

1. Send any request and inspect the **Server** header (**nginx/1.24.0 (Ubuntu)**). On a PHP-backed response the **X-Powered-By** header is also present.

   ![Response headers include the X-Powered-By PHP version](../evidence/Finding-01-RCE/SS-F01-12_rce-uid-www-data.png)
   *__Figure 11.1__ - Response headers include the X-Powered-By PHP version.*

2. Send **PATCH /api/v1/profile/** with **{"email": "test@x.com'"}** and observe the raw MariaDB error in the response body.

   ![Response shows nginx Server header and a raw MariaDB error](../evidence/Finding-02-SQLi/SS-F02-02_sqli-syntax-error.png)
   *__Figure 11.2__ - Response shows nginx Server header and a raw MariaDB error.*

## Business Impact

Knowing the exact versions lets an attacker match published **CVEs** to the
running software without guesswork, which makes finding a working exploit faster. On its own it is
not directly exploitable; it accelerates reconnaissance. Each disclosed component should be
cross-referenced against the **NVD** (https://nvd.nist.gov) for version-specific CVEs as part of
patch management - e.g. the running nginx, PHP, and MariaDB builds should be checked and patched to
the latest stable release.

## Remediation

Suppress version data: set **server_tokens off** in nginx and **expose_php = Off**
in php.ini. Return generic database errors to clients, as covered in Finding 2. Maintain a patch
cadence that tracks NVD advisories for the deployed nginx/PHP/MariaDB versions.

---

[← Finding 10](10-missing-security-headers.md) &nbsp;|&nbsp; [Back to overview](../README.md)
