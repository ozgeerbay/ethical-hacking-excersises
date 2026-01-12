# Idle Scan (Zombie Scan)

## Technique Overview
Idle Scan is a **stealth TCP port scanning technique** that allows an attacker to scan a target **without sending packets directly** to it.
Instead, a third-party host called a **zombie** is used to infer the target’s open ports.

This technique relies on predictable **IP ID (Identification) values** on the zombie system.

---

## Requirements
- Attacker machine (scanner)
- Target machine (to be scanned)
- Zombie host with:
  - Incremental IP ID values
  - Low network traffic (idle)

---

## Lab Scenario
- Attacker: Kali Linux
- Target: 192.168.0.50
- Zombie: 192.168.0.60
- Tool: Nmap

---

## Checking Zombie Suitability
Before performing an idle scan, the zombie must have predictable IP ID behavior.

```bash
nmap --script ipidseq 192.168.0.60

IP ID Sequence Generation: Incremental
```
An incremental IP ID indicates the host is suitable as a zombie.

## Idle Scan Execution
```bash
nmap -sI 192.168.0.60 192.168.0.50
```
-sI → Idle scan mode
192.168.0.60 → Zombie host
192.168.0.50 → Target host

## Example Output

PORT     STATE  SERVICE
21/tcp   open   ftp
22/tcp   open   ssh
80/tcp   open   http

The attacker never sends packets directly to the target.
All traffic appears to originate from the zombie.

How It Works (Short Explanation)

1. The attacker probes the zombie to learn its current IP ID.
2. The attacker sends spoofed SYN packets to the target, pretending to be the zombie.
3. If the target port is open, it responds to the zombie with SYN/ACK.
4. The zombie replies with RST, incrementing its IP ID.
5. The attacker probes the zombie again and observes the IP ID increase.

The difference in IP ID values reveals whether the target port is open or closed.