# Lab 02 – SSH Brute-Force Pattern Identification

## Objective

Identify repeated SSH authentication failures that may indicate brute-force activity, credential guessing, or automated username probing.

This lab moves beyond simply observing authentication logs. The goal is to extract patterns from repeated failed login events and identify suspicious source IP behavior.

---

## Detection Focus

This lab focuses on the following SSH attack pattern:

~~~text
same source IP → repeated failed SSH authentication attempts
~~~

Primary detection signals:

| Signal | Meaning |
|---|---|
| `Failed password` | SSH authentication attempt failed |
| `Invalid user` | Attempt targeted a non-existent account |
| Repeated source IP | Same system generated multiple failures |
| Short time window | Failures occurred close together |
| Multiple usernames | Possible username enumeration or credential probing |

---

## Environment

| Component | Value |
|---|---|
| Server OS | Ubuntu Server 24.04.3 LTS |
| Service | OpenSSH Server (`sshd`) |
| Client / Source | Windows 11 host |
| Source IP | `192.168.56.1` |
| Target IP | `192.168.56.101` |
| Log Source | `/var/log/auth.log` |

Evidence:

![VM IP Address](../screenshots/01-vm-ip-address.png)

![SSH Service Status](../screenshots/02-ssh-service-status.png)

---

## Attack Simulation

Controlled failed SSH login attempts were generated from the Windows host using PowerShell.

Example commands:

~~~powershell
ssh fakeuser1@192.168.56.101
ssh admin@192.168.56.101
ssh test@192.168.56.101
ssh root@192.168.56.101
ssh fakeuser2@192.168.56.101
~~~

These commands generated failed password events and invalid username events in `/var/log/auth.log`.

Evidence:

![Controlled Failed SSH Attempts](../screenshots/05-controlled-failed-ssh-attempts.png)

---

## Procedure

The brute-force pattern identification process followed this workflow:

1. Generated multiple failed SSH authentication attempts
2. Reviewed `/var/log/auth.log` for failed password events
3. Reviewed invalid username attempts
4. Extracted source IP addresses from failed login events
5. Aggregated failed attempts by source IP
6. Identified repeated failures from `192.168.56.1`
7. Treated repeated failures from one source as brute-force-style behavior

---

## Failed Password Evidence

Command used:

~~~bash
sudo grep "sshd" /var/log/auth.log | grep "Failed password" | tail -20
~~~

Purpose:

- Confirm failed SSH authentication events were generated
- Identify target usernames
- Identify source IP addresses
- Validate that attack simulation produced usable telemetry

Evidence:

![Auth Log Failed Password Events](../screenshots/06-auth-log-failed-password-events.png)

---

## Invalid User Evidence

Command used:

~~~bash
sudo grep "sshd" /var/log/auth.log | grep "Invalid user" | tail -20
~~~

Purpose:

- Identify username probing
- Confirm attempts against non-existent accounts
- Separate invalid-user activity from normal password mistakes

Evidence:

![Auth Log Invalid User Events](../screenshots/07-auth-log-invalid-user-events.png)

---

## Source IP Aggregation

The main pattern-identification command counted failed SSH attempts by source IP.

Command used:

~~~bash
sudo grep "Failed password" /var/log/auth.log | grep -oP 'from \K[0-9]+\.[0-9]+\.[0-9]+\.[0-9]+' | sort | uniq -c | sort -nr
~~~

Evidence:

![Failed Logins Aggregated by IP](../screenshots/08-failed-logins-aggregated-by-ip.png)

This showed repeated failed authentication attempts originating from:

~~~text
192.168.56.1
~~~

---

## Why This Command Was Used

A basic field-based command such as:

~~~bash
awk '{print $NF}'
~~~

can be unreliable for OpenSSH logs because the final field may be:

~~~text
ssh2
~~~

instead of the source IP address.

The improved command extracts the IP address after the `from` keyword:

~~~bash
grep -oP 'from \K[0-9]+\.[0-9]+\.[0-9]+\.[0-9]+'
~~~

This is more reliable because OpenSSH authentication logs consistently place the source IP after `from`.

---

## Detection Logic

Brute-force behavior can be identified with the following logic:

~~~text
For each Failed password event:
    Extract source IP
    Count failures per source IP
    If one IP generates repeated failures:
        Flag as suspicious brute-force behavior
~~~

This lab used a simple count-based approach.

The detection does not yet enforce a time window or automatically block the source. Those improvements are added in later labs.

---

## Detection Rule

~~~text
Alert or investigate when:
- A single source IP generates repeated Failed password events
- OR a single source IP probes multiple invalid usernames
- OR failed attempts occur close together in time
~~~

In this lab, the source IP `192.168.56.1` generated repeated failed SSH authentication events.

---

## Observed Attack Pattern

The observed behavior included:

- Multiple failed SSH login attempts
- Multiple invalid username attempts
- Repeated activity from the same source IP
- Authentication attempts against common usernames such as `admin`, `test`, and `root`

This pattern is consistent with brute-force-style SSH activity or credential probing.

---

## Analyst Interpretation

A single failed login is not enough to assume malicious activity.

However, repeated failed logins from the same source IP become more suspicious when combined with:

- invalid usernames
- common default usernames
- multiple attempts in a short period
- repeated failures from one source
- attempts against privileged accounts such as `root`

From a SOC perspective, this type of activity would justify further investigation or automated threshold-based response.

---

## Limitations

This lab uses simple source IP aggregation.

Current limitations:

- No time-window logic
- No automatic blocking
- No severity scoring
- No distinction between valid-user failures and invalid-user failures
- No user baseline
- No SIEM correlation
- No distributed brute-force detection

These limitations are intentional. The purpose of this lab is to identify the basic brute-force pattern before adding mitigation and detection-as-code in later labs.

---

## How This Supports Later Labs

This lab provides the pattern that later controls build on:

| Later Lab | How This Lab Supports It |
|---|---|
| Lab 03 – Fail2Ban Mitigation | Uses repeated failures as the condition for automated blocking |
| Lab 04 – Log Triage | Uses failed and invalid events as investigation evidence |
| Lab 05 – Automated Failed Login Analysis | Automates the source IP aggregation workflow |
| Lab 06 – Time-Window Detection | Adds time-based logic to repeated failed attempts |
| Lab 07 – Failed Attempts Followed by Success | Correlates failed activity with successful authentication |

---

## Outcome

This lab demonstrated that repeated SSH authentication failures can be identified directly from `/var/log/auth.log`.

By filtering failed login events and aggregating by source IP, the lab identified brute-force-style behavior from `192.168.56.1`.

This established the detection basis for automated mitigation, scripted analysis, and time-windowed detection logic in later labs.

---

## Key Takeaway

Brute-force detection starts by finding repetition.

Raw failed login events become useful when they are grouped by source IP, username, and time. This lab proved that Linux authentication logs contain enough information to identify suspicious SSH authentication patterns.
