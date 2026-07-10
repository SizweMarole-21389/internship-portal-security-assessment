[← Back to overview](../README.md)

# Finding 4: Stored Cross-Site Scripting in the Surname Field

**Severity:** High &nbsp;|&nbsp; **CVSS v3.1:** 6.1 (`AV:N/AC:L/PR:L/UI:R/S:C/C:L/I:L/A:N`) · engagement severity High
**CWE:** CWE-79 - Improper Neutralization of Input During Web Page Generation
**OWASP Top 10 (2021):** A03:2021 Injection
**Proof captured:** Yes (marker value redacted)

## Description

The **surname** field accepted by **POST /api/v1/auth/register** is stored
without sanitisation and returned in profile responses without output encoding. A registration
WAF blocks common payloads such as **script**, **img onerror**, and **svg onload** by tag and
event-handler blocklist. An **h4** element with **tabindex=0** and an **onfocus** handler
bypasses the blocklist: the **h4** tag is not blocked, **tabindex=0** makes it focusable, and
**onfocus** fires JavaScript when it gains focus. The payload
`<h4 id=x tabindex=0 onfocus=alert(document.cookie)>`, used here only to confirm script
execution, was stored and returned unencoded.

## Reproduction Steps

1. Register a new account with the bypass payload in the surname field:

   ```json
   {
     "email": "tester@example.com",
     "password": "Sizwe-Test-1234",
     "name": "sizwe",
     "surname": "<h4 id=x tabindex=0 onfocus=alert(document.cookie)>",
     "nickname": "tester1",
     "degree": "Postgraduate in Information Technology",
     "university": "Central University of Technology"
   }
   ```

2. The server returns **201 Created**.

   ![h4 bypass payload accepted at registration with 201](../evidence/Finding-04-XSS/SS-F04-10_xss-h4-registration-201.png)
   *__Figure 4.3__ - H4 bypass payload accepted at registration with 201.*

   ![Confirmation of the accepted payload](../evidence/Finding-04-XSS/SS-F04-11_xss-h4-registration-201-2.png)
   *__Figure 4.4__ - Confirmation of the accepted payload.*

3. Log in and fetch the profile **GET /api/v1/profile/**. The raw payload appears in the
   **surname** field.

   ![Baseline script-tag payload behaviour](../evidence/Finding-04-XSS/SS-F04-04_xss-script-tag-profile-response.png)
   *__Figure 4.2__ - Baseline script-tag payload behaviour.*

   ![Payload returned unencoded in the profile API response](../evidence/Finding-04-XSS/SS-F04-15_xss-h4-profile-api.png)
   *__Figure 4.5__ - Payload returned unencoded in the profile API response.*

4. Open the profile in the web UI. The surname is reflected without output encoding, unlike the
   other fields, confirming the stored payload is served back to the browser.

   ![Surname reflected unencoded; other fields entity-encoded](../evidence/Finding-04-XSS/SS-F04-03_xss-field-comparison-browser.png)
   *__Figure 4.1__ - Surname reflected unencoded; other fields entity-encoded.*

   ![Stored payload rendered unencoded in the profile form](../evidence/Finding-04-XSS/SS-F04-19_xss-patch-profile-browser.png)
   *__Figure 4.6__ - Stored payload rendered unencoded in the profile form.*

## Business Impact

The surname is stored without sanitisation and returned without output
encoding, while the other profile fields are encoded. An attacker can store HTML and script in
their own record, and it is served to anyone who later views that profile, including the
administrator. Where the value is rendered as HTML rather than inside a form input, the onfocus
payload runs in the viewer's session. The application returns access and refresh tokens in the
login response body and keeps them in browser storage that JavaScript can read (see Finding 6),
so a script running in an administrator's session can read both tokens and exfiltrate them to a
server the attacker controls. That gives account takeover, and through the refresh-token flaw in
Finding 6, lasting access. The proof-of-concept only calls `alert(document.cookie)` to confirm
execution; since the tokens are not in cookies, a real attack would read them from browser
storage instead. The WAF gave false confidence - blocking well-known payloads while a trivial
variant walked straight through.

## Remediation

Strip HTML from name fields on input and allow only letters, spaces, hyphens,
and apostrophes. Apply context-aware output encoding when rendering user data. Treat the WAF as
a supplementary control only; blocklists are routinely bypassed.

---

[← Finding 3](03-idor.md) &nbsp;|&nbsp; [Back to overview](../README.md) &nbsp;|&nbsp; [Next: Finding 5 - Assignment Logic Flaws →](05-assignment-logic-flaws.md)
