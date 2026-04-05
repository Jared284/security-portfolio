# Lab 05 – Automated SSH Failed Login Analysis

## Objective

Automate the identification and aggregation of failed SSH authentication attempts to replicate analyst triage previously performed manually.

This lab demonstrates how simple scripting can scale log analysis and quickly identify suspicious authentication patterns.

---

## Environment

- Ubuntu Server with OpenSSH enabled
- Log source: `/var/log/auth.log`
- Analysis performed using a custom shell script
- Prior manual triage methodology established in Lab 04

---

## Method

SSH authentication logs were parsed to extract source IP addresses associated with failed login attempts.

Failures were then aggregated and ranked by frequency to identify the most suspicious sources.

This automation replicates analyst triage logic by focusing on **behavior patterns rather than individual log entries**.

---

## Script Used

Example script used to identify repeated failed SSH authentication attempts:

```bash
grep "Failed password" /var/log/auth.log | \
awk '{print $(NF-3)}' | \
sort | uniq -c | sort -nr
```

This pipeline performs the following steps:

1. Filters authentication failures from the log
2. Extracts the source IP address
3. Counts occurrences by IP
4. Sorts results by frequency

---

## Example Output

```
6 192.168.56.101
2 192.168.56.105
1 192.168.56.110
```

This output indicates that the IP address `192.168.56.101` generated the highest number of failed authentication attempts.

---

## Key Result

The automated analysis successfully identified repeated SSH authentication failures originating from a single source IP.

The detected pattern matched indicators previously observed during manual triage in Lab 04.

---

## Outcome

Manual SSH log investigation from Lab 04 was translated into a repeatable automated workflow.

This reduces reliance on manual log inspection while preserving analyst visibility into suspicious authentication activity.

---

## Defensive Insight

Automating log aggregation allows defenders to rapidly identify high-frequency authentication failures that may indicate brute-force activity.

Script-based analysis helps bridge the gap between raw telemetry and actionable detection signals.
