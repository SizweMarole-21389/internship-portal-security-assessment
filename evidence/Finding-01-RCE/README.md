# Finding 1 — Unrestricted File Upload → Remote Code Execution

> Redacted evidence screenshots for this finding. Flag values, the target domain, credentials, tokens, and personal data are blurred. See the [full report](../../REPORT.md) for context.

### 1. Baseline CV upload accepted

![Baseline CV upload accepted](SS-F01-02_pdf-uploaded-profile.png)

### 2. Server returns 201 Created on upload

![Server returns 201 Created on upload](SS-F01-03_pdf-upload-201.png)

### 3. Profile API returns the stored file UUID

![Profile API returns the stored file UUID](SS-F01-04_profile-uuid-response.png)

### 4. Uploaded file reachable at /cv-view/ with no auth

![Uploaded file reachable at /cv-view/ with no auth](SS-F01-05_cvview-pdf-accessible.png)

### 5. shell.php upload accepted with 201

![shell.php upload accepted with 201](SS-F01-10_shell-php-confirmed.png)

### 6. Response headers include the X-Powered-By PHP version

![Response headers include the X-Powered-By PHP version](SS-F01-12_rce-uid-www-data.png)

### 7. Rce flag first read

![Rce flag first read](SS-F01-13_rce-flag-first-read.png)

### 8. Directory listing via the webshell

![Directory listing via the webshell](SS-F01-14_rce-cvview-ls-la.png)

### 9. /etc/passwd read through the webshell

![/etc/passwd read through the webshell](SS-F01-15_rce-etc-passwd.png)

### 10. Listing of /var/www via the webshell

![Listing of /var/www via the webshell](SS-F01-16_rce-var-www-ls.png)

### 11. Rce flag browser

![Rce flag browser](SS-F01-17_rce-flag-browser.png)

### 12. Rce flag value

![Rce flag value](SS-F01-19_rce-flag-value.png)
