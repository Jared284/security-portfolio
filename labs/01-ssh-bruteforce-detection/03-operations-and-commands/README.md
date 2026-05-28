# 03 – Operations and Commands

## Purpose

This document captures the operational commands used to monitor, triage, investigate, and validate SSH authentication activity inside the Ubuntu Server VM.

This section is focused on analyst workflow, not initial setup.

The goal is to show how raw Linux authentication telemetry can be investigated using standard command-line tools before building automated detection logic.

---

## Operational Workflow

The operational workflow for this lab follows this sequence:

~~~text
Confirm services are running
        ↓
Generate or observe SSH activity
        ↓
Review authentication logs
        ↓
Filter failed, invalid, and accepted logins
        ↓
Aggregate suspicious source IPs
        ↓
Validate Fail2Ban response
        ↓
Run detection scripts
        ↓
Interpret results as a SOC analyst
~~~

This mirrors a practical triage flow: verify the environment, inspect telemetry, identify suspicious patterns, and validate response.

---

## Service Health Checks

### Check SSH Service Status

~~~bash
sudo systemctl status ssh
~~~

Purpose:

- Confirm OpenSSH is installed
- Confirm SSH is actively running
- Validate that the VM can receive SSH authentication attempts
- Troubleshoot connectivity issues

Expected result:

~~~text
active (running)
~~~

Evidence:

![SSH Service Status](../screenshots/02-ssh-service-status.png)

---

### Check Fail2Ban Service Status

~~~bash
sudo systemctl status fail2ban
~~~

Purpose:

- Confirm Fail2Ban is installed
- Confirm Fail2Ban is running
- Validate that automated response tooling is available

Expected result:

~~~text
active (running)
~~~

Evidence:

![Fail2Ban Service Status](../screenshots/03-fail2ban-service-status.png)

---

## Network and Host Validation

### Check VM IP Address

~~~bash
ip a
~~~

Purpose:

- Confirm the Ubuntu VM's host-only adapter IP
- Validate that the target IP is available for SSH testing
- Confirm the VM is reachable from the Windows host

Target VM IP used in this lab:

~~~text
192.168.56.101
~~~

Evidence:

![VM IP Address](../screenshots/01-vm-ip-address.png)

---

## Authentication Log Review

The primary log source for this lab is:

~~~text
/var/log/auth.log
~~~

This file records SSH authentication events including failed logins, invalid usernames, accepted logins, and source IP information.

---

### View Recent SSH Authentication Events

~~~bash
sudo grep "sshd" /var/log/auth.log | tail -30
~~~

Purpose:

- Show recent SSH-related events
- Reduce noise from unrelated authentication activity
- Confirm that OpenSSH is writing logs correctly

---

### View Failed SSH Password Attempts

~~~bash
sudo grep "sshd" /var/log/auth.log | grep "Failed password" | tail -20
~~~

Purpose:

- Identify failed SSH login attempts
- Confirm brute-force or password guessing activity
- Extract usernames and source IP addresses involved in failed authentication

Evidence:

![Auth Log Failed Password Events](../screenshots/06-auth-log-failed-password-events.png)

---

### View Invalid Username Attempts

~~~bash
sudo grep "sshd" /var/log/auth.log | grep "Invalid user" | tail -20
~~~

Purpose:

- Identify username enumeration behavior
- Detect attempts against non-existent accounts
- Support investigation of credential probing activity

Evidence:

![Auth Log Invalid User Events](../screenshots/07-auth-log-invalid-user-events.png)

---

### View Successful SSH Logins

~~~bash
sudo grep "sshd" /var/log/auth.log | grep "Accepted password" | tail -20
~~~

Purpose:

- Identify successful authentication events
- Establish normal login baseline
- Support correlation between failed and successful login activity

---

## Source IP Investigation

### Filter Events by Source IP

~~~bash
sudo grep "192.168.56.1" /var/log/auth.log | tail -30
~~~

Purpose:

- Investigate all authentication activity from the Windows host
- Correlate failed, invalid, and successful events from the same source
- Validate controlled attack simulation behavior

In this lab, the Windows host source IP was:

~~~text
192.168.56.1
~~~

---

### Extract Source IPs from Failed Password Events

~~~bash
sudo grep "Failed password" /var/log/auth.log | grep -oP 'from \K[0-9]+\.[0-9]+\.[0-9]+\.[0-9]+' | sort | uniq -c | sort -nr
~~~

Purpose:

- Extract source IP addresses from failed password events
- Count failed login attempts per source IP
- Identify repeat offenders

Evidence:

![Failed Logins Aggregated by IP](../screenshots/08-failed-logins-aggregated-by-ip.png)

Note:

This regex-based command is preferred over a basic `awk '{print $NF}'` pipeline because the final field in OpenSSH logs can be `ssh2`, not the source IP. Extracting the IP after the `from` keyword is more reliable.

---

## Failed and Successful Login Correlation

### Review Failed, Invalid, and Accepted SSH Events Together

~~~bash
sudo grep "sshd" /var/log/auth.log | grep -E "Failed password|Invalid user|Accepted password" | tail -30
~~~

Purpose:

- Review multiple SSH authentication outcomes together
- Identify suspicious authentication sequences
- Support failed-to-successful login correlation

This command was used to validate the advanced detection case:

~~~text
failed SSH attempts → successful SSH login from same source IP
~~~

Evidence:

![Auth Log Failed Then Successful Login Events](../screenshots/13-auth-log-failed-then-successful-login-events.png)

---

## Fail2Ban Operations

### List All Active Jails

~~~bash
sudo fail2ban-client status
~~~

Purpose:

- Confirm Fail2Ban is running
- Identify active jails
- Verify that the `sshd` jail is enabled

---

### Inspect SSH Jail Status

~~~bash
sudo fail2ban-client status sshd
~~~

Purpose:

- View failed attempt counters
- View current banned IPs
- Confirm Fail2Ban is monitoring SSH activity
- Validate automated blocking behavior

Evidence before attack:

![Fail2Ban SSHD Status Before Attack](../screenshots/04-fail2ban-sshd-status-before-attack.png)

Evidence after attack:

![Fail2Ban Banned IP After Attack](../screenshots/09-fail2ban-banned-ip-after-attack.png)

---

### Check SSH Jail Configuration Values

~~~bash
sudo fail2ban-client get sshd maxretry
sudo fail2ban-client get sshd findtime
sudo fail2ban-client get sshd bantime
~~~

Purpose:

- Confirm the detection threshold
- Confirm the time window
- Confirm the ban duration

Values used in this lab:

| Setting | Value |
|---|---|
| `maxretry` | `5` |
| `findtime` | `600` seconds |
| `bantime` | `600` seconds |

---

### Manually Unban an IP Address

~~~bash
sudo fail2ban-client set sshd unbanip 192.168.56.1
~~~

Purpose:

- Restore SSH access after controlled testing
- Recover from an intentional lab ban
- Continue testing without waiting for the ban timer to expire

This was used during the lab to reset access after validating automated blocking.

---

### Temporarily Adjust Fail2Ban Threshold

~~~bash
sudo fail2ban-client set sshd maxretry 10
~~~

Purpose:

- Prevent Fail2Ban from interfering during failed-to-successful login testing
- Allow failed attempts to occur before a successful login
- Support advanced detection validation

After testing, the threshold was restored:

~~~bash
sudo fail2ban-client set sshd maxretry 5
sudo fail2ban-client get sshd maxretry
~~~

Expected output:

~~~text
5
~~~

---

## Attacker-Side SSH Testing

Commands were run from Windows PowerShell to simulate SSH authentication behavior.

### Failed Login Attempts

~~~powershell
ssh fakeuser1@192.168.56.101
ssh admin@192.168.56.101
ssh test@192.168.56.101
ssh root@192.168.56.101
~~~

Purpose:

- Generate failed password events
- Generate invalid username events
- Trigger Fail2Ban detection thresholds
- Create telemetry for manual and automated analysis

Evidence:

![Controlled Failed SSH Attempts](../screenshots/05-controlled-failed-ssh-attempts.png)

---

### Successful Login

~~~powershell
ssh jared@192.168.56.101
~~~

Purpose:

- Validate legitimate SSH access
- Generate accepted password events
- Establish a successful-login baseline
- Support failed-to-successful login correlation

Evidence:

![Successful SSH Login After Unban](../screenshots/11-successful-ssh-login-after-unban.png)

---

### Validate Blocking After Fail2Ban Ban

~~~powershell
ssh admin@192.168.56.101
~~~

Purpose:

- Confirm Fail2Ban enforcement from the attacker side
- Validate that the banned source IP can no longer reach SSH successfully
- Prove that automated mitigation worked beyond only showing a banned list

Evidence:

![SSH Blocked After Fail2Ban Ban](../screenshots/10-ssh-blocked-after-fail2ban-ban.png)

---

## Detection Script Operations

### Run Failed-Then-Success Detector

~~~bash
sudo python3 failed_then_success_detector.py
~~~

Purpose:

- Parse `/var/log/auth.log`
- Identify failed and invalid SSH attempts
- Correlate those attempts with later successful login activity
- Alert when a suspicious failed-to-successful authentication pattern is found

Evidence:

![Failed Then Success Detector Alert Output](../screenshots/14-failed-then-success-detector-alert-output.png)

---

### Display Detector Script

~~~bash
sed -n '1,45p' failed_then_success_detector.py
sed -n '46,120p' failed_then_success_detector.py
~~~

Purpose:

- Show the detection logic used in the lab
- Document the regex patterns, timestamp parsing, event correlation, and alert output
- Provide code-level evidence for the detection-as-code component

Evidence:

![Failed Then Success Python Detector Script Top](../screenshots/15a-failed-then-success-python-detector-script-top.png)

![Failed Then Success Python Detector Script Bottom](../screenshots/15b-failed-then-success-python-detector-script-bottom.png)

---

## Operational Notes

- Start with raw logs before relying on automation.
- Use `grep` and `tail` to confirm that expected telemetry exists.
- Validate detection output against the original log entries.
- Verify Fail2Ban from both defender and attacker perspectives.
- Restore Fail2Ban thresholds after temporary testing changes.
- Avoid relying on fragile parsing that assumes the source IP is always the final log field.
- Failed-to-successful login correlation is higher-value than failed login counting alone.

---

## Summary

These commands support the full operational workflow for the lab:

~~~text
service validation
network validation
log review
source IP aggregation
Fail2Ban monitoring
automated blocking validation
Python detection execution
analyst interpretation
~~~

The commands demonstrate how a security analyst can move from basic Linux log review into repeatable SSH detection and response operations.
