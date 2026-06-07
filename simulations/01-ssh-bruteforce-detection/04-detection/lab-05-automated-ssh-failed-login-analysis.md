# Lab 05 – Automated SSH Failed Login Analysis

## Objective

Automate the identification and aggregation of failed SSH authentication attempts to replicate analyst triage previously performed manually.

This lab turns manual log review into a repeatable command-line workflow that can quickly identify suspicious source IPs.

The goal is to move from:

~~~text
manual log review
        ↓
source IP extraction
        ↓
failed login aggregation
        ↓
ranked suspicious source list
~~~

---

## Detection Focus

This lab focuses on automating a basic but important detection question:

~~~text
Which source IPs are generating repeated failed SSH login attempts?
~~~

Primary detection pattern:

~~~text
Failed password events grouped by source IP
~~~

This is a foundational detection workflow because repeated failures from one source IP are a common indicator of SSH brute-force activity.

---

## Environment

| Component | Value |
|---|---|
| Server OS | Ubuntu Server 24.04.3 LTS |
| Service | OpenSSH Server (`sshd`) |
| Source IP | `192.168.56.1` |
| Target IP | `192.168.56.101` |
| Log Source | `/var/log/auth.log` |
| Analysis Method | Shell pipeline |
| Prior Step | Manual triage from Lab 04 |

Evidence:

![Auth Log Failed Password Events](../screenshots/06-auth-log-failed-password-events.png)

---

## Why Automate This

Manual log review works for small samples, but it does not scale.

A SOC analyst may need to quickly answer:

- Which IP generated the most failed SSH attempts?
- Are failures isolated or repeated?
- Is one source responsible for most of the suspicious activity?
- Should a source IP be investigated or blocked?
- Did the activity match a brute-force pattern?

Automating the aggregation makes the investigation faster and more consistent.

---

## Manual Triage Logic

Before automation, the analyst workflow looked like this:

1. Open `/var/log/auth.log`
2. Filter for SSH failed password events
3. Identify source IP addresses
4. Count repeated failures
5. Determine whether one IP stood out

This lab converts that workflow into a repeatable command.

---

## Source Data

The source log is:

~~~text
/var/log/auth.log
~~~

Relevant event type:

~~~text
Failed password
~~~

Example SSH failed password pattern:

~~~text
Failed password for invalid user admin from 192.168.56.1 port <port> ssh2
~~~

The key field extracted for aggregation is the source IP after the `from` keyword.

---

## Automated Analysis Command

Command used:

~~~bash
sudo grep "Failed password" /var/log/auth.log | grep -oP 'from \K[0-9]+\.[0-9]+\.[0-9]+\.[0-9]+' | sort | uniq -c | sort -nr
~~~

Evidence:

![Failed Logins Aggregated by IP](../screenshots/08-failed-logins-aggregated-by-ip.png)

---

## Command Breakdown

| Command Stage | Purpose |
|---|---|
| `sudo grep "Failed password" /var/log/auth.log` | Filters the authentication log for failed SSH password attempts |
| `grep -oP 'from \K[0-9]+\.[0-9]+\.[0-9]+\.[0-9]+'` | Extracts the source IP address after the `from` keyword |
| `sort` | Sorts IP addresses so duplicates are grouped |
| `uniq -c` | Counts repeated occurrences per source IP |
| `sort -nr` | Sorts results numerically from highest to lowest |

This produces a ranked list of source IPs responsible for failed SSH authentication attempts.

---

## Why This Version Is More Reliable

A simple field-based command such as:

~~~bash
awk '{print $NF}'
~~~

is not reliable for OpenSSH failed password logs because the final field is often:

~~~text
ssh2
~~~

not the source IP.

A slightly better field-based command may work in some cases:

~~~bash
awk '{print $(NF-3)}'
~~~

However, the regex-based command is clearer and more reliable because it explicitly extracts the IPv4 address after the `from` keyword.

This avoids accidentally counting non-IP fields.

---

## Detection Logic

The automated logic can be described as:

~~~text
For each log line in /var/log/auth.log:
    If the line contains "Failed password":
        Extract the source IP after "from"
        Count the number of failures for that IP

Sort source IPs by total failed attempts.
Investigate IPs with repeated failures.
~~~

This converts noisy raw log entries into an actionable summary.

---

## Example Result

Observed suspicious source:

~~~text
192.168.56.1
~~~

Interpretation:

- The same source IP generated repeated failed SSH login attempts
- The activity matched the brute-force pattern identified in Lab 02
- The same source IP later triggered Fail2Ban mitigation in Lab 03

---

## Detection Rule

~~~text
Investigate when:
- A source IP appears repeatedly in Failed password events
- AND the count is higher than normal baseline behavior
- OR the failures occur with invalid usernames or common admin usernames
~~~

In this lab, repeated failed authentication attempts from `192.168.56.1` were treated as suspicious because they were generated during controlled brute-force simulation.

---

## Relationship to Manual Triage

This lab automates the key triage step from Lab 04.

| Manual Triage Step | Automated Equivalent |
|---|---|
| Read failed login events | `grep "Failed password"` |
| Find source IPs | `grep -oP 'from \\K...'` |
| Count repeated IPs manually | `sort | uniq -c` |
| Rank suspicious sources | `sort -nr` |
| Decide investigation priority | Review highest counts first |

The automation does not replace analyst judgment. It reduces repetitive work so the analyst can focus on interpretation.

---

## Analyst Interpretation

A ranked failed-login summary helps analysts quickly identify concentrated authentication failures.

A high failed-login count from one source IP may indicate:

- brute-force activity
- credential guessing
- misconfigured automation
- stale credentials
- repeated user error
- security testing

The analyst still needs to review context, including:

- whether the usernames were valid or invalid
- whether the source IP is expected
- whether any successful login followed the failures
- whether the activity triggered Fail2Ban
- whether similar activity exists on other hosts

---

## Limitations

This shell-based automation is useful but basic.

Current limitations:

- no time-window logic
- no alert severity
- no distinction between valid and invalid usernames
- no successful login correlation
- no persistent output file
- no enrichment
- no SIEM ingestion
- no distributed attack detection
- no IPv6 parsing

These limitations are addressed in later labs by adding time-window detection and failed-to-successful login correlation.

---

## How This Supports Later Labs

| Later Lab | Connection |
|---|---|
| Lab 06 – Time-Window Detection | Adds time-based thresholding to failed login analysis |
| Lab 07 – Failed Attempts Followed by Success | Correlates failure history with accepted login events |

This lab is intentionally simple because it bridges manual log triage and more advanced detection-as-code.

---

## Outcome

This lab successfully automated failed SSH login aggregation.

The command identified repeated failed authentication activity from `192.168.56.1`, converting raw authentication logs into a ranked investigation output.

This reduced manual review while preserving visibility into the underlying log source.

---

## Key Takeaway

Automation does not need to start with a full SIEM or complex detection platform.

A reliable shell pipeline can turn raw Linux SSH logs into useful detection output when it accurately extracts and aggregates the right fields.
