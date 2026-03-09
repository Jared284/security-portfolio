# Lab 06 – SSH Brute Force Time-Window Detection (Detection-as-Code)

## Objective

Build a time-windowed SSH brute-force detection that generates analyst-readable alerts from Linux authentication logs, upgrading prior whole-file aggregation into operational detection logic.

This lab transitions from retrospective analysis to proactive detection engineering.

---

## Environment

- Ubuntu Server with OpenSSH enabled
- Log source: `/var/log/auth.log`
- Detection logic implemented using a custom Python detection script
- Prior context:
  - Manual SSH log triage (Lab 04)
  - Whole-file automated aggregation (Lab 05)

---

## Threat Model

- **Attacker:** External, unauthenticated remote actor
- **Behavior:** Rapid, repeated SSH authentication attempts using invalid credentials
- **Assumptions:**
  - SSH service is exposed
  - Authentication failures are logged locally
  - Attacker does not possess valid credentials

---

## Detection Rule

**Name:** SSH Brute Force – Failed Password Burst  
**Data Source:** `/var/log/auth.log`  
**Event:** `Failed password`  
**Grouping:** Source IP address  

### Condition

Trigger an alert when failed SSH authentication attempts from a single source IP meet or exceed a defined threshold within a fixed time window.

### Default Parameters

- Threshold: **5 failed attempts**
- Time window: **60 seconds**

This rule is designed to identify burst-style brute-force activity while avoiding alerts on slow, sporadic failures.

---

## Detection Logic Overview

The detection operates as follows:

1. Parse authentication logs and extract:
   - Timestamp
   - Source IP address
   - Targeted username (if present)

2. Normalize timestamps into a comparable format

3. Group failed authentication events by source IP

4. Maintain a sliding time window of failures per IP

5. Trigger an alert when the number of failures within the window reaches the threshold

6. Output a structured alert containing context useful for analyst triage

---

## Detection Script (Python)

```python
from datetime import datetime, timedelta
from collections import defaultdict, deque
import re

LOG_FILE = "/var/log/auth.log"
THRESHOLD = 5
WINDOW_SECONDS = 60

failed_pattern = re.compile(
    r'(?P<month>\w+)\s+(?P<day>\d+)\s+(?P<time>\d+:\d+:\d+).*Failed password.*from (?P<ip>\d+\.\d+\.\d+\.\d+).*for (?P<user>\S+)'
)

MONTH_MAP = {
    "Jan": 1, "Feb": 2, "Mar": 3, "Apr": 4,
    "May": 5, "Jun": 6, "Jul": 7, "Aug": 8,
    "Sep": 9, "Oct": 10, "Nov": 11, "Dec": 12
}

def parse_timestamp(month, day, time_str):
    now = datetime.now()
    hour, minute, second = map(int, time_str.split(":"))
    return datetime(now.year, MONTH_MAP[month], int(day), hour, minute, second)

failures = defaultdict(deque)
usernames = defaultdict(set)

with open(LOG_FILE, "r") as f:
    for line in f:
        match = failed_pattern.search(line)
        if not match:
            continue

        ts = parse_timestamp(
            match.group("month"),
            match.group("day"),
            match.group("time")
        )
        ip = match.group("ip")
        user = match.group("user")

        failures[ip].append(ts)
        usernames[ip].add(user)

        window_start = ts - timedelta(seconds=WINDOW_SECONDS)
        while failures[ip] and failures[ip][0] < window_start:
            failures[ip].popleft()

        if len(failures[ip]) == THRESHOLD:
            print(f"\n[ALERT] SSH Brute Force Detected")
            print(f"Source IP       : {ip}")
            print(f"Failed Attempts : {len(failures[ip])}")
            print(f"Time Window     : {window_start} -> {ts}")
            print(f"Usernames Tried : {', '.join(usernames[ip])}")
```

---

## Running the Detection

The detection script can be executed locally on the server to analyze SSH authentication logs.

```bash
python3 ssh_bruteforce_detector.py
```

The script scans `/var/log/auth.log` and prints an alert when the defined threshold of failed authentication attempts occurs within the configured time window.

---

## Test Evidence

### Test Case 1 — Brute-force burst (alert expected)

**Observed pattern**

- Five or more failed SSH login attempts  
- Same source IP  
- Occurring within approximately 60 seconds  

**Result**

- Detection triggers an alert
- Alert includes source IP, failure count, time window, and targeted usernames

---

### Example Alert Output

```
[ALERT] SSH Brute Force Detected
Source IP       : 192.168.56.101
Failed Attempts : 5
Time Window     : 2026-03-08 14:21:01 -> 2026-03-08 14:21:57
Usernames Tried : root, admin, ubuntu, test
```

This alert provides key context needed for analyst triage.

---

### Test Case 2 — Slow failures over time (no alert)

**Observed pattern**

- Failed logins from the same IP  
- Attempts spread across several minutes  

**Result**

- No alert triggered
- Failures fall outside the defined time window

This validates that the detection differentiates burst activity from benign behavior.

---

## Analyst Notes: False Positives and Validation

### Potential False Positives

- System administrators mistyping credentials
- Misconfigured automation or monitoring scripts

### Validation Steps

- Review source IP reputation and geolocation
- Check historical authentication behavior from the same IP
- Correlate with successful login events
- Confirm whether the IP is associated with known administrative access

---

## Outcome

This lab advances prior work by introducing time-based correlation and explicit alerting logic.

**Progression from Lab 05**

- Whole-file aggregation → sliding time-window detection
- Pattern identification → deterministic alert rule
- Informational output → actionable alert

The result is a reusable detection artifact aligned with real-world SOC workflows.

---

## Future Extensions

- Translate detection logic into SIEM queries (Splunk SPL, Elastic DSL, Sentinel KQL)
- Feed confirmed malicious IPs into Fail2Ban or firewall blocklists
- Add enrichment such as ASN or geolocation data
- Introduce severity tiers based on failure volume
