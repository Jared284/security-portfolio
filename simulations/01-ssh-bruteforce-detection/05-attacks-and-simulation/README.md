# 05 – Attack Simulation

## Purpose

This section documents the controlled SSH attack simulations used to generate real authentication telemetry for the Lab 1 detection workflow.

Rather than relying on fake sample logs, the lab generated actual SSH authentication events against an Ubuntu Server VM and then used those events for detection engineering.

Primary telemetry source:

~~~text
/var/log/auth.log
~~~

The simulations supported:

- SSH authentication log analysis
- brute-force pattern identification
- Fail2Ban automated mitigation
- SOC-style log triage
- failed login aggregation
- time-window detection
- failed attempts followed by successful login correlation

---

## Lab Environment

| Component | Value |
|---|---|
| Attack Source | Windows 11 host |
| Source IP | `192.168.56.1` |
| Target System | Ubuntu Server 24.04.3 LTS VM |
| Target IP | `192.168.56.101` |
| Target Service | OpenSSH Server (`sshd`) |
| Network Type | VirtualBox host-only network |
| Log Source | `/var/log/auth.log` |
| Defensive Tool | Fail2Ban |

All attack activity was performed inside a controlled local lab network.

The SSH target was not exposed to the public internet.

---

## Simulation Workflow

The attack simulation workflow followed this sequence:

~~~text
Windows PowerShell SSH attempts
        ↓
Ubuntu OpenSSH authentication processing
        ↓
Events written to /var/log/auth.log
        ↓
Manual log review
        ↓
Fail2Ban response validation
        ↓
Shell/Python detection logic
        ↓
Alert evidence captured
~~~

This created a full detection engineering loop from attacker behavior to defender visibility.

---

## Scenario 1 – Invalid Username Probing

Attackers often try common usernames to discover valid SSH accounts.

Example usernames used:

- `admin`
- `test`
- `root`
- `fakeuser1`
- `fakeuser2`

Example command:

~~~powershell
ssh admin@192.168.56.101
~~~

Expected log behavior:

~~~text
Invalid user admin from 192.168.56.1
Failed password for invalid user admin from 192.168.56.1
~~~

Evidence:

![Auth Log Invalid User Events](../screenshots/07-auth-log-invalid-user-events.png)

Detection value:

- identifies username probing
- shows attempts against non-existent accounts
- supports brute-force and credential probing analysis

Used in:

- Lab 01 – SSH Authentication Logging
- Lab 02 – SSH Brute-Force Pattern Identification
- Lab 04 – SSH Authentication Log Triage
- Lab 07 – Failed SSH Attempts Followed by Successful Login

---

## Scenario 2 – Repeated Failed SSH Authentication

Repeated failed SSH attempts were generated from Windows PowerShell.

Example commands:

~~~powershell
ssh fakeuser1@192.168.56.101
ssh admin@192.168.56.101
ssh test@192.168.56.101
ssh root@192.168.56.101
ssh fakeuser2@192.168.56.101
~~~

Attacker-side evidence:

![Controlled Failed SSH Attempts](../screenshots/05-controlled-failed-ssh-attempts.png)

Server-side evidence:

![Auth Log Failed Password Events](../screenshots/06-auth-log-failed-password-events.png)

Detection value:

- creates failed password telemetry
- creates repeated source IP activity
- supports brute-force pattern detection
- provides input for Fail2Ban threshold testing

Used in:

- Lab 02 – SSH Brute-Force Pattern Identification
- Lab 03 – Fail2Ban Automated SSH Brute-Force Mitigation
- Lab 04 – SSH Authentication Log Triage
- Lab 05 – Automated SSH Failed Login Analysis
- Lab 06 – SSH Brute Force Time-Window Detection

---

## Scenario 3 – Failed Login Aggregation by Source IP

After failed SSH events were generated, the logs were aggregated by source IP.

Command used:

~~~bash
sudo grep "Failed password" /var/log/auth.log | grep -oP 'from \K[0-9]+\.[0-9]+\.[0-9]+\.[0-9]+' | sort | uniq -c | sort -nr
~~~

Evidence:

![Failed Logins Aggregated by IP](../screenshots/08-failed-logins-aggregated-by-ip.png)

Detection value:

- groups failed login attempts by source IP
- identifies repeat offenders
- converts raw logs into analyst-readable output
- avoids fragile parsing that accidentally extracts `ssh2` instead of the IP

Used in:

- Lab 02 – SSH Brute-Force Pattern Identification
- Lab 05 – Automated SSH Failed Login Analysis

---

## Scenario 4 – Fail2Ban Threshold Trigger

Repeated failed SSH attempts were generated quickly enough to trigger the Fail2Ban `sshd` jail.

Fail2Ban settings used:

| Setting | Value |
|---|---|
| `maxretry` | `5` |
| `findtime` | `600` seconds |
| `bantime` | `600` seconds |

Expected behavior:

~~~text
5 failed SSH attempts within 600 seconds → source IP banned
~~~

Command used to validate ban status:

~~~bash
sudo fail2ban-client status sshd
~~~

Evidence:

![Fail2Ban Banned IP After Attack](../screenshots/09-fail2ban-banned-ip-after-attack.png)

Detection value:

- proves Fail2Ban detected repeated failed authentication
- confirms the source IP was added to the banned list
- validates automated defensive response

Used in:

- Lab 03 – Fail2Ban Automated SSH Brute-Force Mitigation
- Lab 04 – SSH Authentication Log Triage

---

## Scenario 5 – SSH Blocked After Ban

After Fail2Ban banned the source IP, another SSH attempt was made from Windows PowerShell.

Command used:

~~~powershell
ssh admin@192.168.56.101
~~~

Expected behavior:

~~~text
SSH connection times out while source IP is banned.
~~~

Evidence:

![SSH Blocked After Fail2Ban Ban](../screenshots/10-ssh-blocked-after-fail2ban-ban.png)

Detection value:

- validates blocking from the attacker side
- proves the ban was actually enforced
- confirms response effectiveness beyond only checking Fail2Ban status

Used in:

- Lab 03 – Fail2Ban Automated SSH Brute-Force Mitigation
- Lab 04 – SSH Authentication Log Triage

---

## Scenario 6 – Successful SSH Login Baseline

A successful SSH login was performed after unbanning the source IP.

Command used:

~~~powershell
ssh jared@192.168.56.101
~~~

Evidence:

![Successful SSH Login After Unban](../screenshots/11-successful-ssh-login-after-unban.png)

Detection value:

- confirms normal SSH access works
- creates an `Accepted password` event
- establishes successful authentication baseline
- supports later failed-to-successful correlation

Used in:

- Lab 01 – SSH Authentication Logging
- Lab 04 – SSH Authentication Log Triage
- Lab 07 – Failed SSH Attempts Followed by Successful Login

---

## Scenario 7 – Failed Attempts Followed by Successful Login

This was the advanced simulation added to test a higher-value authentication detection.

Sequence generated:

~~~text
failed SSH login as admin
        ↓
failed SSH login as test
        ↓
successful SSH login as jared
~~~

Commands used:

~~~powershell
ssh admin@192.168.56.101
ssh test@192.168.56.101
ssh jared@192.168.56.101
~~~

Attacker-side evidence:

![Failed Then Successful SSH Login Sequence](../screenshots/12-failed-then-successful-ssh-login-sequence.png)

Server-side log evidence:

![Auth Log Failed Then Successful Login Events](../screenshots/13-auth-log-failed-then-successful-login-events.png)

Detection value:

- tests a possible credential compromise pattern
- shows failures and success from the same source IP
- moves beyond basic failed-login counting
- supports SOC-style investigation of suspicious successful access

Used in:

- Lab 07 – Failed SSH Attempts Followed by Successful Login

---

## Scenario 8 – Python Detector Alert Validation

After generating the failed-to-successful login pattern, a Python detector was executed against `/var/log/auth.log`.

Command used:

~~~bash
sudo python3 failed_then_success_detector.py
~~~

Evidence:

![Failed Then Success Detector Alert Output](../screenshots/14-failed-then-success-detector-alert-output.png)

Detection value:

- validates custom detection logic
- proves the detector parsed real authentication logs
- confirms the script correlated failed/invalid events with successful login
- generates analyst-readable alert output

Used in:

- Lab 07 – Failed SSH Attempts Followed by Successful Login

---

## Detection Script Evidence

The Python detection script was captured in two screenshots.

Evidence:

![Failed Then Success Python Detector Script Top](../screenshots/15a-failed-then-success-python-detector-script-top.png)

![Failed Then Success Python Detector Script Bottom](../screenshots/15b-failed-then-success-python-detector-script-bottom.png)

Detection value:

- documents the detection-as-code logic
- shows regex-based parsing of SSH authentication events
- shows time-window correlation
- supports recruiter/interviewer review of the actual detection implementation

---

## Telemetry Generated

The simulations generated the following telemetry types:

| Telemetry Type | Purpose |
|---|---|
| `Invalid user` | Detect username probing |
| `Failed password` | Detect failed authentication |
| `Accepted password` | Detect successful authentication |
| Source IP | Correlate activity by origin |
| Target username | Identify accounts being attempted |
| Timestamp | Support event ordering and time-window detection |
| Fail2Ban ban status | Validate automated mitigation |
| SSH timeout after ban | Validate attacker-side enforcement |

---

## Relationship to Detection Labs

| Attack Behavior | Detection Lab |
|---|---|
| Baseline SSH logging | Lab 01 |
| Invalid username probing | Lab 01, Lab 02, Lab 04, Lab 07 |
| Repeated failed login attempts | Lab 02, Lab 03, Lab 04, Lab 05, Lab 06 |
| Source IP aggregation | Lab 02, Lab 05 |
| Fail2Ban threshold trigger | Lab 03, Lab 04 |
| Attacker-side blocked SSH validation | Lab 03, Lab 04 |
| Successful SSH login baseline | Lab 01, Lab 04, Lab 07 |
| Failed attempts followed by successful login | Lab 07 |
| Python detector alert validation | Lab 07 |

This shows the full lab lifecycle:

~~~text
Attack Simulation
        ↓
Log Generation
        ↓
Manual Investigation
        ↓
Pattern Identification
        ↓
Automated Mitigation
        ↓
Detection-as-Code
        ↓
Alert Validation
~~~

---

## Safety and Containment

All simulations were performed in a host-only VirtualBox lab network.

Safety controls:

- no public exposure of the SSH target
- controlled Windows host used as the only source
- Ubuntu VM isolated from external attackers
- test usernames used intentionally
- Fail2Ban unban/reset performed after testing
- no third-party systems targeted

This kept the lab safe while still producing real SSH authentication telemetry.

---

## Limitations

The simulations were intentionally scoped.

Current limitations:

- single source IP
- single target host
- no distributed brute-force simulation
- no password list tooling
- no SIEM ingestion
- no external threat intelligence
- no post-login command monitoring
- no lateral movement simulation
- no multi-host correlation

These limitations are acceptable because the purpose was to generate controlled SSH telemetry for host-based detection engineering.

---

## Future Enhancements

Future simulations could include:

- low-and-slow SSH failures
- password spraying across multiple local users
- distributed attempts from multiple lab hosts
- post-login command monitoring after successful authentication
- scripted attack generation for repeatability
- SIEM log ingestion and replay
- Splunk, Elastic, or Sentinel query validation
- IP reputation or geolocation enrichment

---

## Outcome

The attack simulations successfully generated real SSH authentication telemetry for every Lab 1 detection stage.

The simulations supported:

- baseline SSH logging
- brute-force pattern identification
- Fail2Ban automated mitigation
- SOC-style log triage
- failed login aggregation
- time-window detection
- failed-to-successful login correlation

---

## Key Takeaway

Detection engineering is stronger when detections are tested against behavior that was actually generated.

This section proves that the Lab 1 detections are based on real SSH activity, real Linux logs, and validated response behavior instead of unsupported example text.
