[← Back to overview](../README.md)

# Finding 6: JWT Refresh Token Not Invalidated on Logout

**Severity:** Medium &nbsp;|&nbsp; **CVSS v3.1:** 5.3 (`AV:N/AC:H/PR:L/UI:N/S:U/C:H/I:N/A:N`)
**CWE:** CWE-613 - Insufficient Session Expiration
**OWASP Top 10 (2021):** A07:2021 Identification and Authentication Failures
**Proof captured:** N/A

## Description

On login the application issues an access token and a refresh token. Logout
**DELETE /api/v1/auth/logout** adds the access token's **jti** to a blocklist, so that token is
rejected afterwards, but the refresh token's **jti** is not blocklisted. A refresh token captured
before logout still works after logout and can be submitted to **POST /api/v1/auth/refresh** to
mint a new access token. Logout does not fully terminate the session.

## Reproduction Steps

1. Log in **POST /api/v1/auth/login** and record both tokens.

   ![Login issues both access and refresh tokens](../evidence/Finding-06-JWT/SS-F06-01_login-response-tokens.png)
   *__Figure 6.1__ - Login issues both access and refresh tokens.*

   ![Decoded token showing the jti and type claims](../evidence/Finding-06-JWT/SS-F06-02_jwt-decoded.png)
   *__Figure 6.2__ - Decoded token showing the jti and type claims.*

2. Log out **DELETE /api/v1/auth/logout** with the access token.

   ![DELETE logout request with the access token](../evidence/Finding-06-JWT/SS-F06-03_logout-request.png)
   *__Figure 6.3__ - DELETE logout request with the access token.*

3. Confirm the access token is now rejected: any authenticated request returns **401 TOKEN_REVOKED**.

   ![Reused access token returns 401 TOKEN_REVOKED](../evidence/Finding-06-JWT/SS-F06-05_access-token-blocked-401.png)
   *__Figure 6.4__ - Reused access token returns 401 TOKEN_REVOKED.*

4. Send the retained refresh token to **POST /api/v1/auth/refresh**.
5. The server returns a new, usable access token despite the logout.

   ![Refresh token still mints a new access token after logout](../evidence/Finding-06-JWT/SS-F06-06_refresh-token-new-access.png)
   *__Figure 6.5__ - Refresh token still mints a new access token after logout.*

## Business Impact

A captured refresh token - for example one stolen through the stored XSS in
Finding 4 - keeps working for the attacker after the real user has logged out, and the user has no
way to end their own session. This is the persistence step in the attack chain: one stolen token is
no longer a one-time problem; it gives lasting access that survives the user logging out.

## Remediation

Blocklist both the access and refresh token **jti** on logout. The
**token_blocklist** table already exists, so add one INSERT for the refresh token's **jti** in the
same logout request.

---

[← Finding 5](05-assignment-logic-flaws.md) &nbsp;|&nbsp; [Back to overview](../README.md) &nbsp;|&nbsp; [Next: Finding 7 - Weak Password Policy →](07-weak-password-policy.md)
