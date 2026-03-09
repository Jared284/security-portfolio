# Detection Engineering – SSH Threats

This section focuses on **SOC-style detection engineering** using native Linux authentication logs to identify, analyze, and respond to common SSH-based attack patterns observed in real environments.

The goal of these labs is to progress from:

```
Raw Telemetry → Detection Logic → Investigation → Automation → Detection-as-Code
```

This mirrors how security teams develop and refine detections in real-world SOC environments.

---

## Detection Scope

All detections in this section rely on analysis of SSH authentication activity recorded in:

```
/var/log/auth.log
```

Primary telemetry sources include:

- OpenSSH authentication events
- Failed login attempts
- Invalid username probes
- Repeated authentication failures from the same source IP
- Burst-style login attempts within short time windows

Controlled attack simulations were used to generate realistic authentication logs for analysis.

---

## Detection Engineering Progression

The labs intentionally follow a step-by-step detection development workflow:

```
Log Visibility
↓
Pattern Identification
↓
Automated Mitigation
↓
Analyst Investigation
↓
Automation of Analysis
↓
Time-Window Detection Logic
```

Each lab builds on the previous one to demonstrate how detection logic evolves from simple observation to operational security controls.

---

## Labs Overview

### Lab 01 – SSH Authentication Logging

Observe how successful and failed SSH login attempts are recorded in authentication logs.

Key focus:

- Understanding log structure
- Identifying authentication outcomes
- Establishing baseline behavior

---

### Lab 02 – SSH Brute-Force Pattern Identification

Identify suspicious authentication patterns by analyzing repeated login failures.

Key focus:

- Aggregating authentication failures
- Identifying brute-force indicators
- Differentiating malicious activity from normal login mistakes

---

### Lab 03 – Fail2Ban Automated SSH Brute-Force Mitigation

Deploy Fail2Ban to automatically block IP addresses generating repeated authentication failures.

Key focus:

- Log-based detection
- Automated defensive response
- Threshold-based blocking

---

### Lab 04 – SSH Authentication Log Triage

Simulate SOC-style investigation of suspicious authentication activity.

Key focus:

- Log investigation techniques
- Correlating authentication events
- Distinguishing attacker behavior from user error

---

### Lab 05 – Automated SSH Failed Login Analysis

Translate manual log triage into a repeatable automated workflow.

Key focus:

- Script-based log parsing
- Aggregating failed authentication attempts
- Identifying suspicious source IPs automatically

---

### Lab 06 – SSH Brute Force Time-Window Detection (Detection-as-Code)

Implement a time-windowed detection rule that generates alerts when repeated authentication failures occur within a short interval.

Key focus:

- Time-based correlation
- Detection rule development
- Alert generation using custom Python logic

This lab introduces the concept of **Detection-as-Code**, where detection logic is implemented programmatically rather than manually.

---

## Skills Demonstrated

These labs demonstrate practical blue-team capabilities including:

- Linux authentication log analysis
- Detection engineering fundamentals
- SOC-style investigation workflows
- Brute-force attack identification
- Defensive automation
- Script-based log analysis
- Time-windowed detection logic
- Security tooling usage (OpenSSH, Fail2Ban, Python)

---

## Why This Matters

SSH brute-force attacks remain one of the most common threats targeting exposed Linux systems.

Security teams must be able to:

- Detect suspicious authentication activity
- Investigate login behavior using system logs
- Identify brute-force attack patterns
- Deploy automated mitigation controls
- Develop repeatable detection logic

These labs demonstrate the hands-on skills required to **detect, investigate, and respond to authentication-based attacks using real system telemetry**.

---

## Future Improvements

Potential expansions for this lab environment include:

- Detection tuning and false-positive reduction
- SIEM rule translation (Splunk SPL, Elastic DSL, Sentinel KQL)
- Alert enrichment with IP reputation or geolocation
- Multi-host attack simulation
- Integration with centralized logging platforms
- Expansion into additional authentication attack scenarios
