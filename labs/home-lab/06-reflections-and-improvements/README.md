# 06 – Reflections and Improvements

## Purpose
Capture lessons learned, operational friction, and analytical insights gained across SSH-focused security labs.

This section documents **judgment development**: what worked, what failed, and how detection thinking evolved across iterations.

---

## Lab 1 Reflection – SSH Authentication Logging

### What I Learned
- Linux records SSH authentication attempts in /var/log/auth.log
- Successful and failed logins are clearly distinguishable
- Invalid user attempts are logged differently from failed passwords for valid users

### What Was Confusing Initially
- Distinguishing between host OS (Windows) and guest OS (Ubuntu)
- Understanding that SSH was initiated from Windows but executed and logged on the Ubuntu server

### What Clicked
- Logs are the primary source of truth for detecting access attempts
- Even failed authentication attempts provide valuable security signal

### Improvements for Next Labs
- Become faster navigating logs using grep and journalctl
- Practice identifying patterns across multiple login attempts rather than individual events

---

## Lab 2 Reflection – Authentication Failure Patterns

This lab demonstrated that repeated authentication failures are significantly more meaningful than isolated failed logins when identifying malicious behavior.

Key takeaway:
- Analysts prioritize **frequency and repetition**, not single failures

---

## Lab 3 Reflection – Automated SSH Defense (Fail2Ban)

### What I Learned
- Log-based detection depends on correct network reachability
- Fail2Ban correlates events across time, not individual failures
- systemd backend is required for SSH log monitoring on Ubuntu 24.04

### What Went Wrong Initially
- Network adapter configuration prevented SSH traffic from reaching the server
- Detection logic appeared broken when the root cause was connectivity

### Improvements for Next Labs
- Validate network connectivity before testing security controls
- Add safeguards to prevent self-lockout during automated enforcement

---

## Lab 4 Reflection – Log Noise vs Signal

- Raw logs are noisy and difficult to interpret at scale
- Aggregation reveals intent that individual log entries obscure
- Effective analysis focuses on **patterns**, not single events

---

## Overall Analyst Takeaways

- Detection begins with raw logs, not tools
- Context and aggregation matter more than isolated indicators
- Automation should follow understanding, not replace it
- Operational mistakes are part of building reliable detection logic

These reflections inform future tuning, automation, and expansion of the lab environment.
