# Linux Home Lab – Detection & Response

This directory contains a Linux-based home lab designed to simulate **SOC-style detection and response workflows** on a single Linux host.

The focus is SSH authentication visibility, brute-force detection, analyst-style triage, and automated mitigation. All work is documented as reproducible, analyst-readable Markdown.

---

## Objectives

- Gain hands-on experience with Linux authentication logs
- Detect SSH brute-force behavior from raw logs
- Perform analyst-style triage and assessment
- Implement automated response controls
- Document detection logic clearly and professionally

---

## Directory Structure

home-lab/  
├── 01-architecture   Lab design, assumptions, and system layout  
├── 02-setup     Linux system setup and security configuration  
├── 03-operations   Day-to-day operational and monitoring commands  
├── 04-detection   Core detection and analysis labs  
├── 05-attacks    Controlled attack simulations  
└── 06-reflections  Lessons learned and future improvements  

---

## Detection Labs (Core Focus)

All primary detection work lives in:

04-detection/

The detection labs progress as follows:

- **Lab 01 – SSH Authentication Logging**  
  Establish baseline visibility into SSH authentication events.

- **Lab 02 – SSH Brute-Force Pattern Identification**  
  Identify repeated authentication failures and group activity by source IP.

- **Lab 03 – Automated SSH Brute-Force Mitigation (Fail2Ban)**  
  Automatically block offending IPs after threshold violations.

- **Lab 04 – SSH Authentication Log Triage**  
  Perform analyst-style triage to distinguish brute-force activity from user error.

- **Lab 05 – Automated SSH Failed Login Analysis**  
  Automate manual SSH log triage into a repeatable detection workflow.

---

## Documentation Standards

- One lab per Markdown file
- Clear objectives, findings, and outcomes
- Evidence-based conclusions
- Detection logic understandable without screenshots

---

## Intended Audience

This lab is designed for:

- SOC analysts
- Detection engineers
- Blue team practitioners
- Students building hands-on security portfolios
