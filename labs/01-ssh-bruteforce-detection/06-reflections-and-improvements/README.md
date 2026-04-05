# 06 – Reflections and Improvements

## Purpose

Capture lessons learned, operational friction, and analytical insights gained while building and testing the SSH detection lab environment.

This section focuses on **analyst judgment development**: what worked, what failed, and how detection thinking evolved across the labs.

---

## Lab 1 Reflection – SSH Authentication Logging

### Key Observations

- Linux records SSH authentication activity in `/var/log/auth.log`
- Successful and failed login attempts are clearly distinguishable
- Attempts against invalid usernames are logged differently from failed passwords for valid users

### Initial Confusion

- Distinguishing between the host system (Windows) and the guest system (Ubuntu)
- Understanding that SSH connections originate from the Windows host but are executed and logged on the Ubuntu server

### Insight

Logs serve as the primary source of truth for authentication activity.

Even failed login attempts generate valuable telemetry that can be used to detect suspicious behavior.

### Improvements for Future Analysis

- Become faster navigating logs using tools such as `grep`
- Focus on identifying patterns across multiple events rather than individual log entries

---

## Lab 2 Reflection – Authentication Failure Patterns

Repeated authentication failures are significantly more meaningful than isolated login failures when identifying potential malicious behavior.

### Key Insight

Security analysts prioritize **frequency and repetition of events**, not single authentication failures.

Aggregation by source IP makes brute-force behavior much easier to identify.

---

## Lab 3 Reflection – Automated SSH Defense (Fail2Ban)

### Key Observations

- Fail2Ban converts log-based detection signals into automated defensive controls
- Authentication failures are correlated across a time window rather than evaluated individually
- Ubuntu 24.04 requires the `systemd` backend for proper Fail2Ban log monitoring

### Operational Issue Encountered

Initial tests suggested the detection logic was not working.

The root cause was incorrect network adapter configuration, which prevented SSH traffic from reaching the server.

### Lesson Learned

Security controls cannot function if underlying telemetry is missing.

Validating **network connectivity and log generation** should always occur before testing detection logic.

---

## Lab 4 Reflection – Log Noise vs Signal

Raw authentication logs contain large volumes of data that are difficult to interpret manually.

### Key Observations

- Individual failed login attempts are low-signal events
- Aggregating authentication failures reveals attacker intent
- Patterns across time and source IP provide stronger indicators of malicious behavior

### Analyst Insight

Detection engineering often begins with **manual investigation**, where analysts identify patterns before translating them into automation.

---

## Lab 5 Reflection – Automating Log Analysis

Manual investigation performed in earlier labs was converted into a repeatable automated workflow.

### Key Observations

- Simple scripting can significantly accelerate log triage
- Automation allows analysts to identify suspicious IPs quickly without scanning raw logs
- Aggregation logic used by analysts can often be translated directly into scripts

### Key Insight

Automation should **augment analyst workflows**, not replace them.

Understanding the underlying log behavior is critical before building automated analysis tools.

---

## Lab 6 Reflection – Time-Window Detection (Detection-as-Code)

This lab introduced time-based correlation and alert generation.

Instead of simply aggregating failures across an entire log file, detection logic was implemented to identify bursts of authentication failures within a defined time window.

### Key Observations

- Time-based detection helps distinguish automated attacks from sporadic user errors
- Detection logic can be expressed programmatically as **Detection-as-Code**
- Structured alerts provide analysts with faster triage context

### Analyst Insight

Effective detection engineering requires moving from:

```
retrospective analysis
→ automated detection
→ actionable alerts
```

This progression mirrors how security teams operationalize detection logic in production environments.

---

## Overall Analyst Takeaways

Several core detection engineering principles emerged during this lab project:

- Detection begins with **raw telemetry**, not security tools
- Aggregation and correlation provide stronger signals than individual events
- Automation should follow understanding, not replace it
- Network and logging visibility are prerequisites for effective detection
- Security controls must be validated through realistic attack simulation

---

## Future Improvements

Potential expansions to this lab environment include:

- Translating detection logic into SIEM queries (Splunk SPL, Elastic DSL, Sentinel KQL)
- Enriching alerts with geolocation or IP reputation data
- Simulating distributed brute-force attacks from multiple IP addresses
- Introducing credential stuffing or password spraying scenarios
- Centralizing authentication logs for cross-host detection analysis
