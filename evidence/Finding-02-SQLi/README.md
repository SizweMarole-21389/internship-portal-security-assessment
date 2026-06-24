# Finding 2 - Error-Based SQL Injection

> Redacted evidence screenshots for this finding. Flag values, the target domain, credentials, tokens, and personal data are blurred. See the [full report](../../REPORT.md) for context.

### 1. Response shows nginx Server header and a raw MariaDB error

![Response shows nginx Server header and a raw MariaDB error](SS-F02-02_sqli-syntax-error.png)

### 2. extractvalue leaks the MariaDB version

![extractvalue leaks the MariaDB version](SS-F02-03_sqli-version-mariadb.png)

### 3. Schema enumeration via information_schema

![Schema enumeration via information_schema](SS-F02-04_sqli-database-enum.png)

### 4. Table enumeration in the active database

![Table enumeration in the active database](SS-F02-06_sqli-table-enum.png)

### 5. Sqli enum internapp ctf

![Sqli enum internapp ctf](SS-F02-08_sqli-enum-internapp-ctf.png)

### 6. Discovery of the users table

![Discovery of the users table](SS-F02-10_sqli-table-enum-users.png)

### 7. Discovery of the assignments table

![Discovery of the assignments table](SS-F02-11_sqli-table-enum-assignments.png)

### 8. Discovery of the user_answers table

![Discovery of the user_answers table](SS-F02-12_sqli-table-enum-user-answers.png)

### 9. Discovery of the cvs table

![Discovery of the cvs table](SS-F02-13_sqli-table-enum-cvs.png)

### 10. Discovery of the token_blocklist table

![Discovery of the token_blocklist table](SS-F02-14_sqli-table-enum-token-blocklist.png)

### 11. Sqli flag part1

![Sqli flag part1](SS-F02-15_sqli-flag-part1.png)

### 12. Sqli flag part2

![Sqli flag part2](SS-F02-16_sqli-flag-part2.png)
