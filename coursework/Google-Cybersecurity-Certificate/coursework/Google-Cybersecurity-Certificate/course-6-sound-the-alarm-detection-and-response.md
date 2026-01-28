# Course 6: Sound the Alarm – Detection and Response

This course explores how cybersecurity professionals detect, investigate, and respond to security incidents using monitoring tools, logging systems, and incident response processes.

---

## 🚨 What is Detection and Response?

- **Detection** is the process of identifying potential security incidents through alerts, monitoring, and analysis.
- **Response** involves investigating, containing, eradicating, and recovering from incidents.

---

## 🔍 Security Monitoring

- Involves continuously watching systems, networks, and activity to detect abnormal or malicious behavior.
- **Indicators of Compromise (IoCs)** – Signs a system has been breached (e.g., strange file hashes, unexpected logins).
- **Indicators of Attack (IoAs)** – Signs an attack is actively taking place.

---

## 📊 Logs and Logging

- **Logs** are records of system and user activity. They’re critical for detecting and investigating incidents.
- Types of logs:
  - **System logs** – Events like reboots, shutdowns
  - **Application logs** – Events inside programs (errors, crashes)
  - **Security logs** – Login attempts, firewall actions
- Tools like `Syslog`, `Event Viewer`, and `journalctl` are used to review logs.

---

## 🛠️ Detection Tools

- **SIEM (Security Information and Event Management)**:
  - Collects and correlates data from various sources (logs, alerts)
  - Helps detect threats and respond faster
  - Example tools: Splunk, IBM QRadar, Elastic Stack

- **IDS (Intrusion Detection System)**:
  - Monitors traffic and alerts if suspicious activity is detected
  - Types:
    - **NIDS** – Network-based
    - **HIDS** – Host-based

- **SOAR (Security Orchestration, Automation, and Response)**:
  - Automates responses to threats
  - Used in Security Operations Centers (SOCs)

---

## ⚙️ Incident Response Lifecycle (NIST Framework)

1. **Preparation** – Build plans, tools, training
2. **Detection and Analysis** – Identify and investigate threats
3. **Containment, Eradication, Recovery** – Isolate and eliminate threat, restore systems
4. **Post-Incident Activities** – Lessons learned, reporting, process improvement

---

## 🧠 Career Insights

- Incident response is fast-paced and crucial in real-world cybersecurity.
- Understanding logs, detection tools, and structured response methods is key for blue team roles like:
  - SOC Analyst
  - Threat Hunter
  - Incident Responder

---

_Completed as part of the Google Cybersecurity Professional Certificate on Coursera._
