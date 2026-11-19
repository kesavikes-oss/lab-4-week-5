# LAB 4: Active Activity – Web Application Penetration Testing

**Name:** KESAVI VIKNESWARAN
**Class:** L01 B02
---

## 1. Target Setup (Victim)

The target used for this penetration testing activity was Damn Vulnerable Web Application (DVWA), an intentionally vulnerable web application deployed using Docker.

**Target Configuration:**

| Component | Configuration |
|---|---|
| Application | DVWA |
| Docker Image | `vulnerables/web-dvwa` |
| Target URL | `http://127.0.0.1:8080` |
| Container Port | 80 |
| Host Port | 8080 |
| Security Level | Low |

The application was hosted locally through Docker. Port 8080 was used because port 80 was already occupied by another local service.

The DVWA security level was configured to **Low** to demonstrate common web application vulnerabilities in a controlled environment.

---

## 2. Own Hosts (Attacker)

The attacker machine used during the activity was a Kali Linux virtual machine.

Kali Linux provided the tools required to perform reconnaissance and test the security of the target application.

**Attacker Configuration:**

| Component | Configuration |
|---|---|
| Operating System | Kali Linux |
| Environment | Virtual Machine |
| Target Address | `127.0.0.1:8080` |
| Tools Used | Nmap, Gobuster, Web Browser |
| Testing Scope | Local DVWA application |

**Tools and Their Purposes:**

- **Nmap:** Used to identify open ports and detect running services.
- **Gobuster:** Used to discover directories and files on the web server.
- **Web Browser:** Used to access DVWA and test its web application vulnerabilities.

---

## 3. Findings, Evidence and Limitations (Interpretation/Analysis)

### 3.1 Reconnaissance Findings

**Nmap Scan**

Nmap identified port `8080/tcp` as open and detected Apache HTTP Server and PHP.

The application also used a PHP session cookie (`PHPSESSID`).

**Evidence:**

![Nmap Scan](evidence/01_nmap_scan.png)

**Gobuster Directory Enumeration**

Gobuster was used to discover accessible directories and files.

The following paths were identified:

- `/config`
- `/docs`
- `/hackable`
- `/phpinfo.php`

The discovery of these paths provided additional information about the application's directory structure. The presence of `phpinfo.php` is noteworthy because it may expose information about the server configuration.

**Evidence:**

![Gobuster Scan](evidence/02_gobuster_scan.png)

### 3.2 Vulnerability Findings

#### A. Command Injection

**Payload Used:**

`127.0.0.1; id`

**Root Cause:**

User-controlled input was passed directly to an operating system command through the `shell_exec()` function without proper input validation or sanitisation.

**Observation:**

The application processed the additional command and returned information about the system user running the web server.

The `id` command revealed that execution occurred under the `www-data` account.

**Impact:**

Command Injection can allow unauthorised operating system command execution through a vulnerable web application. Depending on the permissions of the web server account, this could expose sensitive system information or allow further compromise.

**Evidence:**

![Command Injection](evidence/03_command_injection.png)

#### B. SQL Injection

**Payload Used:**

`' OR '1'='1`

**Root Cause:**

The application constructed SQL queries using direct string concatenation instead of parameterised queries or prepared statements.

This allowed user input to modify the intended SQL query logic.

**Observation:**

The injected condition evaluated to TRUE, causing the application to return records beyond the intended individual user record.

**Impact:**

SQL Injection can allow unauthorised access to database information. Depending on database permissions, an attacker may be able to retrieve, modify or delete sensitive records.

**Evidence:**

![SQL Injection](evidence/04_sql_injection.png)

#### C. Reflected Cross-Site Scripting (XSS)

**Payload Used:**

`<script>alert('XSS-Lab4')</script>`

**Root Cause:**

The application reflected user-controlled input into the HTML response without applying appropriate output encoding.

As a result, the browser interpreted the injected script as executable JavaScript.

**Observation:**

The browser displayed a JavaScript alert containing the message `XSS-Lab4`.

This confirmed that JavaScript execution was possible through the vulnerable input field.

**Impact:**

Reflected XSS can allow attackers to execute unauthorised JavaScript in another user's browser when that user accesses a specially crafted link or request.

Potential consequences include unauthorised actions within the user's session and exposure of accessible page information.

**Evidence:**

![Reflected XSS](evidence/05_reflected_xss.png)

### 3.3 Limitations (Interpretation and Analysis)

Several limitations were identified during this activity.

**Controlled Environment**

Testing was performed against DVWA, an intentionally vulnerable web application configured at the Low security level. Real-world applications may implement additional security measures such as input validation, prepared SQL statements, output encoding and Web Application Firewalls (WAFs).

**Scope Restriction**

Testing was limited to the web application running locally on the host machine. Network-level pivoting, privilege escalation within the Docker container and Denial-of-Service (DoS) testing were outside the scope.

**Wordlist Constraints**

Directory enumeration relied on the `common.txt` wordlist. Therefore, hidden, non-standard or deeply nested endpoints may not have been discovered.

### 3.4 Security Recommendations

Based on the findings, the following security improvements are recommended:

| Vulnerability | Recommended Mitigation |
|---|---|
| Command Injection | Avoid passing user input directly to operating system commands. |
| SQL Injection | Use prepared statements and parameterised queries. |
| Reflected XSS | Implement context-appropriate output encoding. |
| Information Disclosure | Remove unnecessary publicly accessible configuration and information files. |
| Session Security | Configure appropriate HttpOnly, Secure and SameSite cookie attributes. |

### 3.5 Conclusion

The laboratory activity demonstrated the identification and testing of three common web application vulnerabilities: Command Injection, SQL Injection and Reflected Cross-Site Scripting.

Nmap and Gobuster were used to perform reconnaissance, while the DVWA application was used to demonstrate the vulnerabilities in a controlled environment.

The activity highlighted how insecure input handling and insufficient security controls can expose web applications to potential attacks.

Overall, this practical exercise provided experience in reconnaissance, vulnerability identification, evidence collection and security analysis.
