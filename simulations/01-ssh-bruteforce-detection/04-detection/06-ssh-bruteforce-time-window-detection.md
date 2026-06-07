# Lab 06 – SSH Brute Force Time-Window Detection (Detection-as-Code)

## Objective

Build a time-windowed SSH brute-force detection that generates analyst-readable alerts from Linux authentication logs.

This lab upgrades simple whole-file aggregation into detection-as-code by adding time-based correlation.

The goal is to move from:

~~~text
count all failed logins
        ↓
group failures by source IP
        ↓
evaluate failures within a time window
        ↓
generate alert output
~~~

This is closer to how real detection rules work in a SOC or SIEM environment.

---

## Detection Focus

This lab focuses on burst-style SSH brute-force behavior.

Primary detection pattern:

~~~text
same source IP → repeated failed SSH authentication attempts → short time window → alert
~~~

This is different from Lab 05 because Lab 05 counts failures across the whole log file. Lab 06 evaluates whether the failures happen close together in time.

---

## Environment

| Component | Value |
|---|---|
| Server OS | Ubuntu Server 24.04.3 LTS |
| Service | OpenSSH Server (`sshd`) |
| Source IP | `192.168.56.1` |
| Target IP | `192.168.56.101` |
| Log Source | `/var/log/auth.log` |
| Detection Method | Python detection script |
| Detection Type | Time-window correlation |
| Prior Step | Automated aggregation from Lab 05 |

Evidence:

![Auth Log Failed Password Events](../screenshots/06-auth-log-failed-password-events.png)

---

## Threat Model

This detection models an attacker attempting rapid SSH authentication attempts against a Linux server.

Assumptions:

- SSH is reachable from the attack source
- OpenSSH logs failed authentication attempts
- Failed attempts are written to `/var/log/auth.log`
- The attacker generates several failures in a short period
- Repeated failures from one source IP may indicate brute-force behavior

The attacker does not need to successfully authenticate for this detection to fire.

---

## Detection Rule

Rule name:

~~~text
SSH Brute Force – Failed Password Burst
~~~

Data source:

~~~text
/var/log/auth.log
~~~

Event type:

~~~text
Failed password
~~~

Grouping field:

~~~text
Source IP address
~~~

Default condition:

~~~text
Alert when a single source IP generates 5 or more failed SSH password attempts within 60 seconds.
~~~

Default parameters:

| Parameter | Value |
|---|---|
| Failure threshold | `5` failed attempts |
| Time window | `60` seconds |
| Grouping | Source IP |
| Event | `Failed password` |

---

## Why Time Windows Matter

A full-log count can identify which IP failed the most times, but it does not explain how quickly those failures happened.

Example:

| Pattern | Interpretation |
|---|---|
| 5 failures in 30 seconds | More suspicious |
| 5 failures across several days | Less suspicious |
| 1 failure from a known admin | Likely benign |
| Many failures from one unknown IP | Brute-force indicator |

Time-window detection adds context by asking:

~~~text
Did the failures cluster close enough together to indicate attack behavior?
~~~

---

## Raw Log Evidence

Failed SSH attempts were confirmed in `/var/log/auth.log`.

Command used:

~~~bash
sudo grep "sshd" /var/log/auth.log | grep "Failed password" | tail -20
~~~

Evidence:

![Auth Log Failed Password Events](../screenshots/06-auth-log-failed-password-events.png)

---

## Detection Logic Overview

The Python detection logic performs these steps:

1. Open `/var/log/auth.log`
2. Find SSH `Failed password` events
3. Extract:
   - timestamp
   - source IP
   - attempted username
4. Group failed attempts by source IP
5. Maintain a rolling time window for each source IP
6. Trigger an alert when the threshold is reached inside the window
7. Print analyst-readable alert output

Detection flow:

~~~text
Failed password log line
        ↓
Extract timestamp, username, and source IP
        ↓
Store event under source IP
        ↓
Remove events outside the time window
        ↓
Count remaining failures
        ↓
Alert if count >= threshold
~~~

---

## Important Parsing Note

Ubuntu authentication logs can vary depending on logging configuration.

In this lab, the observed logs used ISO-style timestamps similar to:

~~~text
2026-05-28T14:07:49.37315+00:00 ubuntu-server-lab sshd[...]: Failed password for invalid user admin from 192.168.56.1 port ... ssh2
~~~

Because of that, the detector should parse the actual observed format instead of assuming older syslog-style timestamps such as:

~~~text
May 28 14:07:49
~~~

The detection logic also extracts the source IP after the `from` keyword instead of relying on the final log field, because the final field is often `ssh2`.

---

## Python Detection Script

Example detector logic:

~~~python
#!/usr/bin/env python3

import re
from collections import defaultdict, deque
from datetime import datetime, timedelta

AUTH_LOG = "/var/log/auth.log"
FAILURE_THRESHOLD = 5
TIME_WINDOW_SECONDS = 60

failed_attempts = defaultdict(deque)
usernames = defaultdict(set)

failed_pattern = re.compile(
    r"^(?P<timestamp>\\S+)\\s+\\S+\\s+sshd\\[\\d+\\]: Failed password for (?:invalid user )?(?P<user>\\S+) from (?P<ip>\\d+\\.\\d+\\.\\d+\\.\\d+)"
)

def parse_timestamp(timestamp):
    return datetime.strptime(timestamp, "%Y-%m-%dT%H:%M:%S.%f%z")

with open(AUTH_LOG, "r") as log_file:
    for line in log_file:
        match = failed_pattern.search(line)

        if not match:
            continue

        timestamp = parse_timestamp(match.group("timestamp"))
        ip = match.group("ip")
        user = match.group("user")

        failed_attempts[ip].append(timestamp)
        usernames[ip].add(user)

        window_start = timestamp - timedelta(seconds=TIME_WINDOW_SECONDS)

        while failed_attempts[ip] and failed_attempts[ip][0] < window_start:
            failed_attempts[ip].popleft()

        if len(failed_attempts[ip]) == FAILURE_THRESHOLD:
            print("[ALERT] SSH Brute Force Detected")
            print(f"Source IP: {ip}")
            print(f"Failed attempts: {len(failed_attempts[ip])}")
            print(f"Time window: {window_start} -> {timestamp}")
            print(f"Usernames tried: {', '.join(sorted(usernames[ip]))}")
            print()
~~~

---

## Running the Detection

The detector can be executed locally on the Ubuntu server.

Example command:

~~~bash
sudo python3 ssh_bruteforce_time_window_detector.py
~~~

The script scans `/var/log/auth.log` and prints an alert when one source IP reaches the configured failed-login threshold inside the time window.

---

## Expected Alert Output

Example output:

~~~text
[ALERT] SSH Brute Force Detected
Source IP: 192.168.56.1
Failed attempts: 5
Time window: 2026-05-28 14:07:10+00:00 -> 2026-05-28 14:08:04+00:00
Usernames tried: admin, fakeuser1, root, test
~~~

This alert provides the key fields an analyst needs:

- source IP
- failure count
- time window
- attempted usernames

---

## Test Case 1 – Brute-Force Burst

Observed pattern:

- multiple failed SSH login attempts
- same source IP
- attempts occurred close together
- usernames included common or invalid accounts

Expected result:

~~~text
Alert triggered
~~~

Why:

The failure count met or exceeded the threshold inside the detection window.

---

## Test Case 2 – Slow Failures Over Time

Observed pattern:

- failed logins from the same source IP
- attempts spread out over a longer period
- failure count does not meet threshold inside the configured window

Expected result:

~~~text
No alert triggered
~~~

Why:

The activity does not meet the burst threshold, even if the total number of failures is nonzero.

---

## Relationship to Fail2Ban

Fail2Ban and this Python detector both use failed SSH authentication activity, but they serve different purposes.

| Control | Purpose |
|---|---|
| Fail2Ban | Enforces automated blocking |
| Python time-window detector | Produces analyst-readable detection output |

Fail2Ban answers:

~~~text
Should this IP be banned?
~~~

The Python detector answers:

~~~text
What suspicious authentication pattern happened, when did it happen, and which users were targeted?
~~~

---

## Analyst Interpretation

A burst of failed SSH login attempts from one source IP may indicate:

- brute-force attack
- credential guessing
- automated scanning
- password spraying against common accounts
- misconfigured automation repeatedly failing authentication

Analyst review should consider:

- whether the source IP is expected
- which usernames were targeted
- whether the attempts were clustered in time
- whether Fail2Ban also banned the source
- whether a successful login occurred after the failures

The final item is especially important and is expanded in Lab 07.

---

## False Positive Considerations

Potential false positives include:

- administrator mistyping a password repeatedly
- stale credentials in a script
- monitoring system using expired credentials
- repeated SSH attempts from a known management host
- intentional security testing

Tuning options:

- increase the threshold
- adjust the time window
- allowlist known administrative IPs
- alert only on invalid-user attempts
- raise severity for privileged usernames such as `root`
- correlate with successful login events

---

## Limitations

This detector is intentionally lightweight.

Current limitations:

- parses local `/var/log/auth.log` only
- supports the observed ISO-style timestamp format
- does not parse every possible Linux auth log format
- does not enrich IP addresses
- does not persist alert history
- does not send notifications
- does not ingest into a SIEM
- does not detect distributed brute force across many IPs
- does not inspect post-login activity

These limitations are acceptable because the goal is to demonstrate time-window detection logic using real SSH authentication telemetry.

---

## How This Supports Lab 07

Lab 06 detects repeated failures within a time window.

Lab 07 builds on that idea by asking a higher-value question:

~~~text
Did a source IP fail multiple times and then successfully authenticate?
~~~

That makes Lab 07 more compromise-oriented, while Lab 06 remains focused on brute-force burst detection.

---

## Outcome

This lab introduced detection-as-code for SSH brute-force behavior.

The lab demonstrated how to:

- parse SSH authentication logs
- extract timestamps, usernames, and source IPs
- group events by source IP
- evaluate failures inside a time window
- generate analyst-readable alert output

This is a major improvement over whole-file aggregation because it considers event timing, not just total counts.

---

## Key Takeaway

A high failed-login count is useful, but timing makes the signal stronger.

Time-window detection helps distinguish burst-style brute-force behavior from isolated login mistakes or low-frequency noise.
