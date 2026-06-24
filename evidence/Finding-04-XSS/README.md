# Finding 4 - Stored Cross-Site Scripting

> Redacted evidence screenshots for this finding. Flag values, the target domain, credentials, tokens, and personal data are blurred. See the [full report](../../REPORT.md) for context.

### 1. Surname reflected unencoded; other fields entity-encoded

![Surname reflected unencoded; other fields entity-encoded](SS-F04-03_xss-field-comparison-browser.png)

### 2. Baseline script-tag payload behaviour

![Baseline script-tag payload behaviour](SS-F04-04_xss-script-tag-profile-response.png)

### 3. h4 bypass payload accepted at registration with 201

![h4 bypass payload accepted at registration with 201](SS-F04-10_xss-h4-registration-201.png)

### 4. Confirmation of the accepted payload

![Confirmation of the accepted payload](SS-F04-11_xss-h4-registration-201-2.png)

### 5. Payload returned unencoded in the profile API response

![Payload returned unencoded in the profile API response](SS-F04-15_xss-h4-profile-api.png)

### 6. Xss h4 flag profile response

![Xss h4 flag profile response](SS-F04-17_xss-h4-flag-profile-response.png)

### 7. Stored payload rendered unencoded in the profile form

![Stored payload rendered unencoded in the profile form](SS-F04-19_xss-patch-profile-browser.png)
