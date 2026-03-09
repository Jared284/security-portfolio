# 03 – Operations and Commands

## Purpose

Document the day-to-day operational commands used to monitor, triage, and investigate SSH authentication activity on the Ubuntu server.

This section reflects **analyst workflow**, not setup steps.

The goal is to demonstrate how signals are extracted from logs and services during routine security operations.

---

## Common SSH Operations

Check SSH service status:

```
systemctl status ssh
```

This check is used to confirm service availability during troubleshooting or after configuration changes.

---

## Authentication Log Analysis

View full authentication log:

```
sudo cat /var/log/auth.log
```

Identify failed login attempts:

```
sudo grep "Failed password" /var/log/auth.log
```

Identify successful logins:

```
sudo grep "Accepted" /var/log/auth.log
```

These commands provide quick visibility into authentication behavior without requiring external tooling.

---

## IP-Based Filtering

Filter authentication events by source IP:

```
grep "from 192.168.56.101" /var/log/auth.log
```

This is useful for validating activity from known hosts or investigating suspicious source addresses.

---

## fail2ban Operations

Check fail2ban service status:

```
sudo systemctl status fail2ban
```

List all active jails:

```
sudo fail2ban-client status
```

Inspect the SSH jail specifically:

```
sudo fail2ban-client status sshd
```

Manually unban an IP address:

```
sudo fail2ban-client set sshd unbanip <IP>
```

These commands are used to validate automated enforcement and correct false positives.

---

## Failed Login Aggregation

Extract and summarize failed SSH login attempts by source IP:

```
grep "Failed password" /var/log/auth.log | awk '{print $NF}' | sort | uniq -c | sort -nr
```

This pipeline helps identify brute-force patterns by highlighting repeat offenders.

---

## SSH Failed Login Automation

Execute Python script to summarize failed login attempts:

```
python3 ssh_failed_summary.py
```

This script automates log parsing to reduce manual effort during repeated analysis tasks.

---

## Operational Notes

- Authentication logs serve as the primary signal source for SSH activity
- Manual grep-based analysis is used first for transparency and validation
- Automation is layered on top only after understanding raw log behavior
- fail2ban actions are always verified to avoid unintended lockouts

This operational approach mirrors real SOC triage: start simple, validate signals, then automate.
