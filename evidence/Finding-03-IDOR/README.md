# Finding 3 - Insecure Direct Object Reference (IDOR)

> Redacted evidence screenshots for this finding. Flag values, the target domain, credentials, tokens, and personal data are blurred. See the [full report](../../REPORT.md) for context.

### 1. Leaderboard page listing intern nicknames

![Leaderboard page listing intern nicknames](SS-F03-02_leaderboard-page.png)

### 2. Leaderboard API response with full nickname list

![Leaderboard API response with full nickname list](SS-F03-03_idor-leaderboard-nicknames.png)

### 3. Privileged profile returned to an intern token

![Privileged profile returned to an intern token](SS-F03-12_idor-admin-profile-flag.png)

### 4. Another intern's full profile retrieved

![Another intern's full profile retrieved](SS-F03-13_idor-intern001-response.png)

### 5. A further intern profile retrieved

![A further intern profile retrieved](SS-F03-15_idor-intern003-response.png)

### 6. A further intern profile retrieved

![A further intern profile retrieved](SS-F03-16_idor-intern005-response.png)

### 7. Confirms the vulnerability scales across the user base

![Confirms the vulnerability scales across the user base](SS-F03-18_idor-intern085-response.png)
