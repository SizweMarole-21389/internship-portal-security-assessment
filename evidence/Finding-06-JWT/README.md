# Finding 6 - JWT Refresh Token Not Invalidated on Logout

> Redacted evidence screenshots for this finding. Flag values, the target domain, credentials, tokens, and personal data are blurred. See the [full report](../../REPORT.md) for context.

### 1. Login issues both access and refresh tokens

![Login issues both access and refresh tokens](SS-F06-01_login-response-tokens.png)

### 2. Decoded token showing the jti and type claims

![Decoded token showing the jti and type claims](SS-F06-02_jwt-decoded.png)

### 3. DELETE logout request with the access token

![DELETE logout request with the access token](SS-F06-03_logout-request.png)

### 4. Logout success response

![Logout success response](SS-F06-04_logout-success-response.png)

### 5. Reused access token returns 401 TOKEN_REVOKED

![Reused access token returns 401 TOKEN_REVOKED](SS-F06-05_access-token-blocked-401.png)

### 6. Refresh token still mints a new access token after logout

![Refresh token still mints a new access token after logout](SS-F06-06_refresh-token-new-access.png)
