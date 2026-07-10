[← Back to overview](../README.md)

# Finding 2: Error-Based SQL Injection in Profile Update Endpoint

**Severity:** High &nbsp;|&nbsp; **CVSS v3.1:** 8.1 (`AV:N/AC:L/PR:L/UI:N/S:U/C:H/I:L/A:N`)
**CWE:** CWE-89 - Improper Neutralization of Special Elements used in an SQL Command
**OWASP Top 10 (2021):** A03:2021 Injection
**Proof captured:** Yes (marker value redacted)

## Description

The profile update endpoint **PATCH /api/v1/profile/** interpolates the
**email** field directly into a SQL query with no sanitisation. A single quote in the email
value returns a raw MariaDB syntax error in the response, confirming the input reaches the
database and that detailed errors are exposed. The injection supports error-based extraction
with MariaDB's **extractvalue()** function, which embeds the result of a subquery inside an
XPath error message. Output is capped at 32 characters, so longer values are read in chunks.
Using this, the database version, all schema names, table structures, and the contents of
sensitive tables were extracted.

## Reproduction Steps

1. Log in and capture a valid access token.
2. Confirm the injection point with a single quote in the email field:

   ```http
   PATCH /api/v1/profile/
   {"email": "test'@gmail.com"}
   ```

   Expected: **500** with a raw MariaDB syntax error in the response body.

   ![Single quote returns a raw MariaDB syntax error](../evidence/Finding-02-SQLi/SS-F02-02_sqli-syntax-error.png)
   *__Figure 2.1__ - Single quote returns a raw MariaDB syntax error.*

3. Leak the database version:

   ```json
   {"email": "test@x.com' AND extractvalue(1,concat(0x7e,(SELECT @@version)))-- "}
   ```

   Response includes **10.11.14-MariaDB-0ubuntu0.24.04**.

   ![extractvalue leaks the MariaDB version](../evidence/Finding-02-SQLi/SS-F02-03_sqli-version-mariadb.png)
   *__Figure 2.2__ - Extractvalue leaks the MariaDB version.*

4. Enumerate schema and table names by substituting subqueries against **information_schema**.

   ![Schema enumeration via information_schema](../evidence/Finding-02-SQLi/SS-F02-04_sqli-database-enum.png)
   *__Figure 2.3__ - Schema enumeration via information_schema.*

   ![Table enumeration in the active database](../evidence/Finding-02-SQLi/SS-F02-06_sqli-table-enum.png)
   *__Figure 2.4__ - Table enumeration in the active database.*

   ![Discovery of the users table](../evidence/Finding-02-SQLi/SS-F02-10_sqli-table-enum-users.png)
   *__Figure 2.5__ - Discovery of the users table.*

   ![Discovery of the assignments table](../evidence/Finding-02-SQLi/SS-F02-11_sqli-table-enum-assignments.png)
   *__Figure 2.6__ - Discovery of the assignments table.*

   ![Discovery of the user_answers table](../evidence/Finding-02-SQLi/SS-F02-12_sqli-table-enum-user-answers.png)
   *__Figure 2.7__ - Discovery of the user_answers table.*

   ![Discovery of the cvs table](../evidence/Finding-02-SQLi/SS-F02-13_sqli-table-enum-cvs.png)
   *__Figure 2.8__ - Discovery of the cvs table.*

   ![Discovery of the token_blocklist table](../evidence/Finding-02-SQLi/SS-F02-14_sqli-table-enum-token-blocklist.png)
   *__Figure 2.9__ - Discovery of the token_blocklist table.*

5. Extract longer values in 32-character chunks using `SUBSTRING(...)` and recombine.

## Business Impact

The injection reaches the live application database, which holds
**users**, **cvs**, **assignments**, **user_answers**, and **token_blocklist**. The **users**
table contains over 120 records with real email addresses and bcrypt password hashes,
including the administrator hash (redacted):

```
$2b$12$[REDACTED]
```

The **cvs** table exposes every uploaded CV UUID, which combines with the unauthenticated
file serving in Finding 1 to allow download of every candidate's CV. A single unprivileged
request results in a **complete breach of personal data for the entire user base**, along
with the password hashes needed to escalate toward the administrator account offline. Because
the users table holds names, email addresses, and CV references of over 120 real individuals,
exposing it on a single request is exactly the kind of unauthorised access POPIA's breach
provisions address: a failure of the Section 19 security-safeguards duty that triggers the
Section 22 obligation to notify the Information Regulator and affected data subjects.

## Remediation

Use parameterised statements for every query that includes user input,
across the whole codebase, so SQL logic is separated from data. Stop returning raw database
errors to clients; log full error details server-side only.

---

[← Finding 1](01-unrestricted-file-upload.md) &nbsp;|&nbsp; [Back to overview](../README.md) &nbsp;|&nbsp; [Next: Finding 3 - IDOR →](03-idor.md)
