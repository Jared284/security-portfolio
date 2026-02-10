# Security Portfolio – Home Lab & Detection Engineering

This repository documents hands-on cybersecurity labs focused on **log analysis, detection engineering, and automated defensive controls** in a Linux home lab environment.

The objective of this portfolio is to demonstrate defensive security thinking — from raw telemetry to detection, triage, and automated response — in a realistic Linux environment.
---

## Home Lab Overview

This home lab simulates real-world SSH attack scenarios against a Linux server and incrementally builds detection, analysis, and automated mitigation capabilities.

### Focus Areas
- SSH authentication logging
- Brute-force pattern identification
- Analyst-style log triage
- Threshold-based detection logic
- Automated response using Fail2Ban
- Scripted log analysis and aggregation

---

## Detection Labs (Core)

All detection-focused labs are located under:

```text
labs/home-lab/04-detection/
```

### Lab Progression

- **Lab 01 – SSH Authentication Logging**  
  Establish baseline visibility into SSH authentication activity.

- **Lab 02 – SSH Brute-Force Pattern Identification**  
  Identify repeated authentication failures and group activity by source IP.

- **Lab 03 – Automated SSH Brute-Force Mitigation (Fail2Ban)**  
  Detect and automatically block offending IPs based on retry thresholds.

- **Lab 04 – SSH Authentication Log Triage**  
  Perform analyst-style triage to distinguish brute-force activity from user error.

- **Lab 05 – Automated SSH Failed Login Analysis**  
  Automate manual SSH log triage into a repeatable, script-driven detection workflow.

These labs are designed to mirror **SOC-level detection and response workflows**, progressing from raw logs to automated enforcement.

---

## Repository Structure

```text
labs/
└── home-lab/
    ├── 01-architecture      # Lab design and system layout
    ├── 02-setup             # System setup and configuration
    ├── 03-operations        # Day-to-day operational commands
    ├── 04-detection         # Detection and analysis labs (core)
    ├── 05-attacks           # Controlled attack simulations
    └── 06-reflections       # Lessons learned and improvements
```

---

## Coursework

The `coursework/` directory contains structured notes and summaries from formal cybersecurity education (e.g., Google Cybersecurity Certificate).

These materials support foundational knowledge but are **separate from the hands-on labs**, which are the primary focus of this repository.

---

## Skills Demonstrated

- Linux log analysis (`auth.log`)
- SSH security monitoring
- Detection engineering concepts
- Threshold-based alerting
- Fail2Ban configuration and enforcement
- Analyst-style incident triage
- Security automation with scripting
- Clear technical documentation

---

## Status

This repository is actively maintained and reflects ongoing development in **defensive security and detection engineering**.
