# Lab 07 – Failed SSH Attempts Followed by Successful Login

## Objective

Detect a higher-risk SSH authentication pattern where failed or invalid login attempts are followed by a successful login from the same source IP.

This lab expands the detection workflow beyond brute-force failure counting.

The goal is to identify the sequence:

~~~text
failed SSH attempts
        ↓
same source IP
        ↓
successful SSH login
        ↓
possible credential compromise signal
~~~

This pattern matters because an attacker may fail several times before eventually guessing, reusing, or obtaining valid credentials.

---

## Detection Focus

This lab focuses on authentication correlation, not just failure volume.

Primary detection pattern:

~~~text
same source IP → failed/invalid SSH attempts → accepted SSH login
~~~

This is higher-value than basic brute-force detection because the final event is successful access.

---

## Environment

| Component | Value |
|---|---|
| Server OS | Ubuntu Server 24.04.3 LTS |
| Service | OpenSSH Server (`sshd`) |
| Source System | Windows 11 host |
| Source IP | `192.168.56.1` |
| Target IP | `192.168.56.101` |
| Log Source | `/var/log/auth.log` |
| Detection Method | Python log parsing |
| Detection Window | 10 minutes |
| Failure Threshold | 2 failed/invalid events before success |

---

## Why This Detection Matters

A failed login by itself is common.

Repeated failed logins are more suspicious.

But failed logins followed by a successful login from the same source IP are significantly more important because the source may have moved from failed access attempts into valid authenticated access.

This may indicate:

- successful password guessing
- credential stuffing
- use of a reused password
- account takeover
- unauthorized access after repeated failures
- suspicious authentication behavior requiring analyst review

In a SOC environment, this pattern would normally deserve higher priority than failed attempts alone.

---

## Detection Scenario

Controlled SSH authentication activity was generated from the Windows host against the Ubuntu VM.

The sequence included:

1. Failed SSH login attempt using `admin`
2. Failed SSH login attempt using `test`
3. Successful SSH login using the valid `jared` account
4. Review of `/var/log/auth.log`
5. Python-based correlation of failed and successful events

This created a realistic authentication sequence for testing compromise-oriented detection logic.

---

## Attack Simulation

The failed login attempts were generated from Windows PowerShell.

Example commands:

~~~powershell
ssh admin@192.168.56.101
ssh test@192.168.56.101
~~~

After failed attempts, a successful login was performed:

~~~powershell
ssh jared@192.168.56.101
~~~

Evidence:

![Failed Then Successful SSH Login Sequence](../screenshots/12-failed-then-successful-ssh-login-sequence.png)

---

## Raw Log Evidence

The Ubuntu authentication log confirmed that the same source IP generated both failed and successful SSH authentication events.

Command used:

~~~bash
sudo grep "sshd" /var/log/auth.log | grep -E "Failed password|Invalid user|Accepted password" | tail -30
~~~

Observed event sequence:

- `Invalid user admin from 192.168.56.1`
- `Failed password for invalid user admin from 192.168.56.1`
- `Invalid user test from 192.168.56.1`
- `Failed password for invalid user test from 192.168.56.1`
- `Accepted password for jared from 192.168.56.1`

Evidence:

![Auth Log Failed Then Successful Login Events](../screenshots/13-auth-log-failed-then-successful-login-events.png)

---

## Detection Logic

The detector parses `/var/log/auth.log` and tracks three SSH event types:

| Event Type | Log Pattern | Detection Purpose |
|---|---|---|
| Failed password | `Failed password for ... from <IP>` | Identify failed authentication |
| Invalid user | `Invalid user ... from <IP>` | Identify username probing |
| Accepted password | `Accepted password for ... from <IP>` | Identify successful authentication |

The script groups failed and invalid-user events by source IP.

When a successful login occurs, the script checks whether the same source IP generated failed or invalid authentication events during the previous 10 minutes.

If the number of prior failures meets the threshold, the script prints an alert.

---

## Detection Rule

~~~text
Alert when:
- A source IP generates >= 2 failed or invalid SSH authentication events
- AND the same source IP later generates an Accepted password event
- AND the failed or invalid events occurred within 10 minutes before the successful login
~~~

Configured values:

| Parameter | Value |
|---|---|
| Failure threshold | `2` |
| Lookback window | `10 minutes` |
| Grouping field | Source IP |
| Success event | `Accepted password` |

---

## Python Detection Script

The Python script performs the following actions:

1. Reads `/var/log/auth.log`
2. Extracts failed password events
3. Extracts invalid user events
4. Extracts accepted password events
5. Stores failed and invalid events by source IP
6. Looks backward 10 minutes when a successful login occurs
7. Alerts when enough prior failures exist for that same source IP

Evidence:

![Failed Then Success Python Detector Script Top](../screenshots/15a-failed-then-success-python-detector-script-top.png)

![Failed Then Success Python Detector Script Bottom](../screenshots/15b-failed-then-success-python-detector-script-bottom.png)

---

## Alert Output

The detector successfully generated an alert for the failed-to-successful authentication sequence.

Example alert output:

~~~text
[ALERT] Failed SSH attempts followed by successful login
Source IP: 192.168.56.1
Successful user: jared
Failures before success: 4
~~~

Evidence:

![Failed Then Success Detector Alert Output](../screenshots/14-failed-then-success-detector-alert-output.png)

---

## Detection Outcome

The detection successfully identified a suspicious authentication sequence.

Confirmed results:

- failed SSH authentication events were generated from `192.168.56.1`
- invalid username attempts were generated from `192.168.56.1`
- successful SSH login for `jared` occurred from `192.168.56.1`
- the Python detector correlated prior failures with later successful access
- the alert output included source IP, successful user, login time, and failure history

---

## Analyst Interpretation

This alert should be treated as more important than failed login attempts alone.

A SOC analyst reviewing this alert would investigate:

- whether the successful login was expected
- whether the source IP belongs to the legitimate user
- whether the user recognizes the login
- whether the same source tried other usernames
- whether the source IP appears elsewhere in logs
- whether any suspicious post-login activity occurred
- whether the account should be reset, locked, or reviewed

The key concern is not only that failures occurred. The concern is that access eventually succeeded.

---

## Severity Reasoning

This detection can be interpreted as a possible compromise signal.

| Condition | Severity Impact |
|---|---|
| Failed attempts only | Lower severity |
| Failed attempts against invalid users | More suspicious |
| Repeated failures from same IP | Brute-force indicator |
| Successful login after failures | Higher-priority investigation |
| Successful login as privileged user | Critical escalation |
| Post-login suspicious activity | Strong compromise indicator |

In this lab, the activity was controlled. In a real environment, this pattern would justify escalation.

---

## False Positive Considerations

This detection can produce false positives.

Possible benign explanations:

- legitimate user mistyped their username
- legitimate user mistyped their password several times
- administrator tested SSH access
- shared jump host generated mixed authentication activity
- automation used stale credentials before a valid login

Tuning options:

- increase the failure threshold
- shorten or lengthen the lookback window
- allowlist known admin IPs
- add username baselining
- increase severity for privileged usernames
- require multiple distinct usernames before alerting
- correlate with post-login commands

---

## Relationship to Previous Labs

| Previous Lab | Connection to Lab 07 |
|---|---|
| Lab 01 – SSH Authentication Logging | Established failed and successful SSH log patterns |
| Lab 02 – Brute-Force Pattern Identification | Identified repeated failures from one source |
| Lab 03 – Fail2Ban Mitigation | Demonstrated automated blocking for repeated failures |
| Lab 04 – Log Triage | Built analyst workflow for suspicious SSH activity |
| Lab 05 – Automated Failed Login Analysis | Automated source IP aggregation |
| Lab 06 – Time-Window Detection | Added time-based detection logic |
| Lab 07 – Failed Attempts Followed by Success | Correlates failures with successful access |

Lab 07 is the highest-value detection in this sequence because it focuses on possible successful compromise, not only attempted access.

---

## Limitations

This lab uses a controlled local VM environment.

Current limitations:

- single source IP
- single target host
- no SIEM ingestion
- no alert routing
- no GeoIP enrichment
- no ASN enrichment
- no user baseline
- no endpoint telemetry beyond SSH logs
- no post-login command monitoring
- no multi-host correlation
- no distributed attack detection

The goal was to prove the correlation logic using real Linux authentication telemetry.

---

## Future Improvements

Potential improvements include:

- translate the rule into Splunk SPL
- translate the rule into Elastic/KQL
- translate the rule into Microsoft Sentinel KQL
- add post-login command monitoring
- add severity scoring
- enrich source IPs with reputation data
- detect failures against multiple users followed by success
- write alerts to a file or webhook
- integrate with centralized logging

---

## Outcome

This lab successfully expanded the SSH detection workflow from brute-force detection into compromise-oriented authentication correlation.

The final detector identified a source IP that generated failed and invalid SSH attempts before successfully logging in as `jared`.

This demonstrates how raw Linux authentication logs can be converted into higher-fidelity detection logic.

---

## Key Takeaway

Failed login detection is useful, but failed attempts followed by successful login are much more important.

This lab shows how detection engineering can move beyond counting failures and begin identifying authentication sequences that may indicate successful unauthorized access.
