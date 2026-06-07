# Detection Engineering – SSH Threats

## Purpose

This section documents the detection engineering workflow for SSH authentication threats using Linux authentication telemetry.

The detection labs progress from raw SSH log review into automated mitigation, scripted analysis, time-window detection, and failed-to-successful login correlation.

Primary log source:

~~~text
/var/log/auth.log
~~~

This section is the core detection layer of Lab 1.

---

## Detection Engineering Workflow

The detection workflow follows this progression:

~~~text
Raw SSH Authentication Logs
        ↓
Baseline Event Review
        ↓
Brute-Force Pattern Identification
        ↓
Automated Mitigation with Fail2Ban
        ↓
SOC-Style Log Triage
        ↓
Shell-Based Failed Login Aggregation
        ↓
Python Time-Window Detection
        ↓
Failed Attempts Followed by Successful Login Correlation
~~~

Each lab builds on the previous one.

The goal is to show how raw Linux authentication telemetry can become operational detection logic.

---

## Detection Scope

All detections in this section rely on SSH authentication activity generated against the Ubuntu Server VM.

Primary telemetry includes:

| Telemetry | Purpose |
|---|---|
| `Accepted password` | Identifies successful SSH authentication |
| `Failed password` | Identifies failed SSH authentication |
| `Invalid user` | Identifies username probing |
| Source IP | Correlates activity by origin |
| Target username | Shows accounts being attempted |
| Timestamp | Enables event ordering and time-window detection |
| Fail2Ban status | Validates automated mitigation |

Primary source:

~~~text
/var/log/auth.log
~~~

---

## Environment

| Component | Value |
|---|---|
| Target System | Ubuntu Server 24.04.3 LTS |
| Target IP | `192.168.56.101` |
| Attack Source | Windows 11 host |
| Source IP | `192.168.56.1` |
| Service | OpenSSH Server (`sshd`) |
| Defensive Tool | Fail2Ban |
| Detection Methods | Manual triage, shell commands, Python scripts |
| Evidence Folder | `../screenshots/` |

---

## Labs Overview

## Lab 01 – SSH Authentication Logging

File:

~~~text
lab-01-ssh-authentication-logging.md
~~~

Purpose:

Observe how SSH authentication attempts are written to Linux authentication logs.

Key focus:

- reviewing `/var/log/auth.log`
- identifying `Accepted password` events
- identifying `Failed password` events
- identifying `Invalid user` events
- establishing baseline telemetry for later detections

Why it matters:

This lab establishes the raw log source. Detection engineering starts by understanding the telemetry before building rules or automation.

---

## Lab 02 – SSH Brute-Force Pattern Identification

File:

~~~text
lab-02-ssh-bruteforce-pattern-identification.md
~~~

Purpose:

Identify repeated SSH authentication failures that may indicate brute-force behavior.

Key focus:

- repeated failed logins
- invalid username probing
- source IP aggregation
- brute-force-style behavior recognition

Detection pattern:

~~~text
same source IP → repeated failed SSH authentication attempts
~~~

Why it matters:

This lab moves from individual log entries to pattern recognition.

---

## Lab 03 – Fail2Ban Automated SSH Brute-Force Mitigation

File:

~~~text
lab-03-fail2ban-automated-ssh-bruteforce-mitigation.md
~~~

Purpose:

Use Fail2Ban to automatically block a source IP that exceeds the SSH failed-login threshold.

Key focus:

- Fail2Ban `sshd` jail
- threshold-based detection
- automated IP banning
- attacker-side block validation

Detection and response pattern:

~~~text
repeated failed SSH attempts → Fail2Ban threshold exceeded → source IP banned
~~~

Why it matters:

This lab shows how authentication telemetry can trigger automated defensive action.

---

## Lab 04 – SSH Authentication Log Triage

File:

~~~text
lab-04-ssh-authentication-log-triage.md
~~~

Purpose:

Perform SOC-style investigation of suspicious SSH authentication activity.

Key focus:

- failed password review
- invalid username review
- successful login review
- source IP aggregation
- Fail2Ban status validation
- analyst assessment

Why it matters:

This lab demonstrates the analyst workflow behind detection decisions. It shows how to interpret the logs, not just collect them.

---

## Lab 05 – Automated SSH Failed Login Analysis

File:

~~~text
lab-05-automated-ssh-failed-login-analysis.md
~~~

Purpose:

Automate failed SSH login aggregation using a shell pipeline.

Key focus:

- extracting source IPs from failed password events
- counting failures by source IP
- ranking suspicious sources
- replacing manual counting with repeatable analysis

Core command:

~~~bash
sudo grep "Failed password" /var/log/auth.log | grep -oP 'from \K[0-9]+\.[0-9]+\.[0-9]+\.[0-9]+' | sort | uniq -c | sort -nr
~~~

Why it matters:

This lab converts manual triage into a repeatable workflow while keeping the logic transparent.

---

## Lab 06 – SSH Brute Force Time-Window Detection

File:

~~~text
lab-06-ssh-bruteforce-time-window-detection.md
~~~

Purpose:

Implement time-windowed brute-force detection using Python detection-as-code logic.

Key focus:

- parsing SSH failed password events
- extracting timestamp, username, and source IP
- grouping failures by source IP
- evaluating failures inside a time window
- producing analyst-readable alert output

Detection pattern:

~~~text
same source IP → 5 failed SSH attempts → within 60 seconds → alert
~~~

Why it matters:

This lab improves on whole-file aggregation by adding time context. Timing helps separate burst-style brute-force behavior from isolated login mistakes.

---

## Lab 07 – Failed SSH Attempts Followed by Successful Login

File:

~~~text
lab-07-failed-ssh-attempts-followed-by-successful-login.md
~~~

Purpose:

Detect a higher-risk authentication sequence where failed or invalid SSH attempts are followed by successful login from the same source IP.

Key focus:

- failed authentication correlation
- invalid username correlation
- successful login detection
- same-source IP tracking
- possible credential compromise logic

Detection pattern:

~~~text
failed SSH attempts
        ↓
same source IP
        ↓
successful SSH login
~~~

Why it matters:

This is the strongest detection in the section because it identifies possible successful access after suspicious authentication failures.

Failed login attempts are useful. Failed attempts followed by successful login are much more important.

---

## Detection Coverage

| Lab | Detection / Control | Signal | Method | Output |
|---|---|---|---|---|
| Lab 01 | SSH authentication logging | `Accepted password`, `Failed password`, `Invalid user` | Manual log review | Baseline telemetry |
| Lab 02 | Brute-force pattern identification | Repeated failed logins | Source IP aggregation | Suspicious source IP |
| Lab 03 | Fail2Ban mitigation | Failure threshold exceeded | Fail2Ban `sshd` jail | Source IP ban |
| Lab 04 | SSH log triage | Failed, invalid, and accepted events | SOC-style investigation | Analyst assessment |
| Lab 05 | Automated failed login analysis | Failed password events | Shell pipeline | Ranked failed-login sources |
| Lab 06 | Time-window brute-force detection | Failures clustered in time | Python detection-as-code | Alert output |
| Lab 07 | Failed attempts followed by success | Prior failures followed by accepted login | Python correlation | Possible compromise-style alert |

---

## Evidence Map

The detection labs use centralized screenshot evidence from:

~~~text
../screenshots/
~~~

| Evidence | File |
|---|---|
| VM IP address | `01-vm-ip-address.png` |
| SSH service status | `02-ssh-service-status.png` |
| Fail2Ban service status | `03-fail2ban-service-status.png` |
| Fail2Ban SSHD status before attack | `04-fail2ban-sshd-status-before-attack.png` |
| Controlled failed SSH attempts | `05-controlled-failed-ssh-attempts.png` |
| Failed password log events | `06-auth-log-failed-password-events.png` |
| Invalid user log events | `07-auth-log-invalid-user-events.png` |
| Failed logins aggregated by IP | `08-failed-logins-aggregated-by-ip.png` |
| Fail2Ban banned IP after attack | `09-fail2ban-banned-ip-after-attack.png` |
| SSH blocked after Fail2Ban ban | `10-ssh-blocked-after-fail2ban-ban.png` |
| Successful SSH login after unban | `11-successful-ssh-login-after-unban.png` |
| Failed then successful login sequence | `12-failed-then-successful-ssh-login-sequence.png` |
| Failed then successful auth log events | `13-auth-log-failed-then-successful-login-events.png` |
| Failed-then-success detector alert output | `14-failed-then-success-detector-alert-output.png` |
| Python detector script top | `15a-failed-then-success-python-detector-script-top.png` |
| Python detector script bottom | `15b-failed-then-success-python-detector-script-bottom.png` |

---

## Detection Logic Progression

The detection logic becomes more mature across the section:

| Stage | Question Answered |
|---|---|
| Lab 01 | What does SSH authentication telemetry look like? |
| Lab 02 | Which source IPs are failing repeatedly? |
| Lab 03 | Can repeated failures trigger automated blocking? |
| Lab 04 | Is the behavior benign or suspicious? |
| Lab 05 | Can failed login aggregation be automated? |
| Lab 06 | Are failures clustered inside a short time window? |
| Lab 07 | Did failures lead to successful access? |

This progression mirrors real detection engineering: start with visibility, then build correlation and response.

---

## Key Technical Lessons

## 1. Raw Logs Come First

The detection logic depends on reliable SSH telemetry.

Before writing rules or scripts, the lab confirmed that OpenSSH wrote authentication events to:

~~~text
/var/log/auth.log
~~~

---

## 2. Source IP Extraction Must Be Reliable

Simple field parsing can be fragile.

Bad approach:

~~~bash
awk '{print $NF}'
~~~

Problem:

~~~text
The final field in OpenSSH logs can be ssh2 instead of the source IP.
~~~

Improved approach:

~~~bash
grep -oP 'from \K[0-9]+\.[0-9]+\.[0-9]+\.[0-9]+'
~~~

This extracts the source IP based on the `from` keyword.

---

## 3. Automated Blocking Must Be Validated

Fail2Ban status alone is not enough.

Stronger validation includes:

- confirming the source IP appears in the banned list
- attempting SSH again from the attacker side
- confirming the connection times out or cannot complete

---

## 4. Failed-to-Successful Login Correlation Is Higher Value

A source IP that only fails authentication is suspicious.

A source IP that fails authentication and then successfully logs in is more important.

That pattern may indicate possible credential compromise and deserves higher-priority investigation.

---

## Skills Demonstrated

This detection section demonstrates:

- SSH authentication log analysis
- Linux command-line triage
- source IP aggregation
- brute-force pattern identification
- Fail2Ban validation
- attacker-side response testing
- shell-based automation
- Python detection-as-code
- regex-based log parsing
- timestamp parsing
- time-window detection
- failed-to-successful login correlation
- SOC-style analyst reasoning
- false-positive analysis

---

## Limitations

Current limitations:

- single Ubuntu target host
- single source IP
- local-only VirtualBox lab
- no centralized SIEM ingestion
- no distributed brute-force detection
- no post-login command monitoring
- no alert routing
- no IP reputation or GeoIP enrichment
- no persistent alert storage

These limitations are acceptable because this section focuses on host-based SSH detection engineering using real Linux authentication telemetry.

---

## Future Improvements

Potential future improvements include:

- Translate detections into Splunk SPL
- Translate detections into Elastic/KQL
- Translate detections into Microsoft Sentinel KQL
- Add post-login command monitoring
- Simulate low-and-slow SSH failures
- Simulate password spraying across multiple users
- Add multiple attacker IPs for distributed brute-force simulation
- Add severity scoring
- Add known admin IP allowlists
- Forward authentication logs into a SIEM-style platform

---

## Key Takeaway

This section shows the full detection engineering path for SSH authentication threats.

The strongest part of the section is the progression from basic failed login visibility to a higher-value detection for failed SSH attempts followed by successful login.

That final detection moves the lab beyond brute-force monitoring and toward compromise-oriented authentication analysis.
