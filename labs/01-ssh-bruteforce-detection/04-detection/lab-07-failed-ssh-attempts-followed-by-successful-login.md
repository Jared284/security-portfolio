# Lab 07 – Failed SSH Attempts Followed by Successful Login

## Objective

This lab expands the SSH brute-force detection workflow by identifying a higher-risk authentication pattern:

~~~text
multiple failed SSH attempts → successful SSH login from the same source IP
~~~

This pattern is important because a successful login after repeated failures may indicate that an attacker guessed, reused, or obtained valid credentials after unsuccessful attempts.

Unlike basic brute-force detection, this lab does not only look for failed logins. It correlates failed authentication activity with later successful authentication from the same source IP.

---

## Detection Scenario

A Windows host generated controlled SSH authentication attempts against the Ubuntu Server VM.

The sequence included:

1. Failed SSH login attempt using an invalid or unauthorized username
2. Additional failed SSH login attempt from the same source IP
3. Successful SSH login using the valid `jared` account
4. Server-side log review in `/var/log/auth.log`
5. Python-based detection logic to alert on failures followed by success

---

## Environment

| Component | Value |
|---|---|
| Attacker / Source System | Windows 11 host |
| Target System | Ubuntu Server 24.04.3 LTS VM |
| Target IP | `192.168.56.101` |
| Source IP | `192.168.56.1` |
| Log Source | `/var/log/auth.log` |
| Detection Method | Python log parsing |
| Detection Window | 10 minutes |
| Failure Threshold | 2 failed/invalid events before success |

---

## Why This Detection Matters

Basic brute-force detection only identifies repeated failed authentication attempts.

However, a more serious signal occurs when those failures are followed by a successful login.

This may indicate:

- Password guessing that eventually succeeded
- Credential stuffing with a valid reused password
- Account takeover after repeated failed attempts
- Suspicious login behavior requiring analyst review
- A source IP moving from reconnaissance or brute force into authenticated access

In a SOC environment, this pattern would usually deserve higher priority than failed attempts alone because the attacker may have gained access.

---

## Attack Simulation

The attack sequence was generated from Windows PowerShell.

Failed login attempts were made against invalid or unauthorized users:

~~~powershell
ssh admin@192.168.56.101
ssh test@192.168.56.101
~~~

After the failed attempts, a successful login was performed with the valid `jared` account:

~~~powershell
ssh jared@192.168.56.101
~~~

Evidence:

![Failed then successful SSH login sequence](../screenshots/12-failed-then-successful-ssh-login-sequence.png)

---

## Raw Log Evidence

The authentication log confirmed that failed and successful SSH events came from the same source IP.

Command used:

~~~bash
sudo grep "sshd" /var/log/auth.log | grep -E "Failed password|Invalid user|Accepted password" | tail -30
~~~

The log showed:

- `Invalid user admin from 192.168.56.1`
- `Failed password for invalid user admin from 192.168.56.1`
- `Invalid user test from 192.168.56.1`
- `Failed password for invalid user test from 192.168.56.1`
- `Accepted password for jared from 192.168.56.1`

Evidence:

![Auth log failed then successful login events](../screenshots/13-auth-log-failed-then-successful-login-events.png)

---

## Detection Logic

The Python detector parses `/var/log/auth.log` and tracks three SSH authentication event types:

| Event Type | Log Pattern |
|---|---|
| Failed password | `Failed password for ... from <IP>` |
| Invalid user | `Invalid user ... from <IP>` |
| Successful login | `Accepted password for ... from <IP>` |

The script groups failed and invalid-user events by source IP.

When a successful SSH login occurs, the script checks whether that same source IP had at least two failed or invalid-user events within the previous 10 minutes.

If the threshold is met, the script prints an alert.

---

## Detection Rule

~~~text
Alert when:
- Source IP has >= 2 failed or invalid SSH authentication events
- AND the same source IP later has an Accepted password event
- AND the failures occurred within 10 minutes before the success
~~~

---

## Python Detection Script

The script uses regular expressions to parse SSH authentication events and correlate source IP activity over a time window.

Evidence:

![Failed then successful Python detector script top](../screenshots/15a-failed-then-success-python-detector-script-top.png)

![Failed then successful Python detector script bottom](../screenshots/15b-failed-then-success-python-detector-script-bottom.png)

---

## Alert Output

The detector successfully generated alerts for failed SSH attempts followed by successful login.

Example alert output:

~~~text
[ALERT] Failed SSH attempts followed by successful login
Source IP: 192.168.56.1
Successful user: jared
Failures before success: 4
~~~

Evidence:

![Failed then success detector alert output](../screenshots/14-failed-then-success-detector-alert-output.png)

---

## Detection Outcome

This lab successfully detected a suspicious authentication pattern where failed SSH attempts were followed by a successful login from the same source IP.

The detection confirmed:

- Failed and invalid SSH events were generated from `192.168.56.1`
- A successful login for `jared` occurred from the same source IP
- The Python script correlated failures and success within a defined time window
- The alert output identified the source IP, successful user, login time, and failure history

---

## Analyst Interpretation

This event pattern should be treated as more suspicious than failed login attempts alone.

A SOC analyst reviewing this alert would investigate:

- Whether the successful login was expected
- Whether the source IP belongs to a known user system
- Whether the account owner recognizes the login
- Whether similar attempts occurred against other accounts
- Whether post-login activity occurred after authentication

In a real environment, this detection could support escalation into account compromise investigation.

---

## False Positive Considerations

This detection can generate false positives in cases such as:

- A legitimate user mistyping their password before logging in successfully
- A user attempting to log in with the wrong username first
- Admins testing SSH access from a known management host
- Shared jump boxes where multiple users authenticate from the same source IP

To reduce false positives, the detection could be tuned by adding:

- Higher failure thresholds
- Longer or shorter time windows
- Known administrator IP allowlists
- User-account baselining
- Severity scoring based on username, source IP, and number of failures

---

## Limitations

This lab uses a controlled local VM environment, so all authentication attempts came from a single host-only source IP.

The detection does not yet include:

- GeoIP enrichment
- ASN enrichment
- User-agent or device fingerprinting
- Multi-host correlation
- SIEM ingestion
- Post-login command monitoring
- Automated alert routing

The goal of this lab was to prove the detection logic using real Linux authentication telemetry.

---

## Skills Demonstrated

This lab demonstrates:

- Linux authentication log analysis
- SSH attack simulation
- Detection engineering
- Python log parsing
- Regex-based event extraction
- Time-window correlation
- Failed-to-successful authentication detection
- SOC-style alert interpretation
- False-positive analysis

---

## Key Takeaway

Failed SSH attempts are useful, but failed attempts followed by successful login are much more important.

This lab demonstrates how raw Linux authentication logs can be transformed into higher-value detection logic that identifies possible credential compromise behavior.
