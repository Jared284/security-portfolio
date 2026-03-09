
---

## Detection Labs

The detection section contains six labs demonstrating the evolution of detection logic.

### Lab 01 – SSH Authentication Logging

Understand how SSH authentication attempts are recorded in Linux authentication logs.

Focus:
- log structure  
- successful vs failed login events  

---

### Lab 02 – SSH Brute-Force Pattern Identification

Identify suspicious authentication patterns by aggregating repeated failed login attempts.

Focus:
- brute-force indicators  
- source IP aggregation  

---

### Lab 03 – Fail2Ban Automated SSH Mitigation

Deploy Fail2Ban to automatically block IP addresses responsible for repeated authentication failures.

Focus:
- log-based detection  
- automated response  

---

### Lab 04 – SSH Authentication Log Triage

Simulate analyst investigation of suspicious authentication activity.

Focus:
- log analysis  
- attacker behavior identification  

---

### Lab 05 – Automated SSH Failed Login Analysis

Automate log aggregation using scripting to replicate analyst triage workflows.

Focus:
- script-based log parsing  
- automated detection support  

---

### Lab 06 – SSH Brute-Force Time Window Detection (Detection-as-Code)

Implement a time-windowed detection rule that generates alerts when repeated authentication failures occur within a short interval.

Focus:
- time-based detection  
- detection-as-code  
- alert generation  

---

## Skills Demonstrated

This project demonstrates practical blue-team skills including:

- Linux log analysis  
- authentication log investigation  
- brute-force attack detection  
- SOC-style triage workflows  
- defensive automation  
- script-based log analysis  
- time-window detection logic  
- security tooling usage (OpenSSH, Fail2Ban, Python)

---

## Why This Project Matters

Detection engineering often begins with **raw system telemetry**, not security tools.

Understanding how attackers interact with systems and how those interactions appear in logs is essential for building effective detections.

This project demonstrates how detection logic can evolve from:
