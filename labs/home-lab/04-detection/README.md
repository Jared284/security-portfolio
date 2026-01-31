\# Detection Engineering – SSH Threats



This section focuses on \*\*SOC-style detection engineering\*\* using native Linux logs to identify, analyze, and respond to common SSH-based attack patterns observed in real environments.



The goal of these labs is to move from \*\*raw telemetry → detection logic → triage and mitigation\*\*, mirroring how blue teams operate in practice.



---



\## Detection Scope



All detections in this section are based on analysis of:



\- `/var/log/auth.log`

\- OpenSSH authentication events

\- Repeated failed authentication attempts

\- Indicators of brute-force activity



The lab environment intentionally simulates attacker behavior to generate realistic logs for analysis.



---



\## Labs Overview



\### \*\*Lab 01 – SSH Authentication Logging\*\*

\- Observes how successful and failed SSH logins are recorded

\- Identifies key log fields (user, source IP, timestamp)

\- Establishes baseline behavior for normal vs abnormal access



\### \*\*Lab 02 – SSH Brute-Force Pattern Identification\*\*

\- Detects repeated failed login attempts from a single IP

\- Differentiates between benign mistakes and malicious patterns

\- Introduces threshold-based detection logic



\### \*\*Lab 03 – Fail2Ban Automated SSH Brute-Force Mitigation\*\*

\- Implements Fail2Ban to automatically block malicious IPs

\- Maps detections to defensive controls

\- Demonstrates prevention as an extension of detection



\### \*\*Lab 04 – SSH Authentication Log Triage\*\*

\- Investigates suspicious login activity

\- Correlates failed and successful attempts

\- Simulates analyst triage and escalation decisions



\### \*\*Lab 05 – Automated SSH Failed Login Analysis\*\*

\- Uses scripting to parse authentication logs

\- Extracts failed login counts and source IPs

\- Demonstrates how automation scales manual analysis



---



\## Skills Demonstrated



\- Linux log analysis

\- Detection engineering fundamentals

\- SOC-style investigative thinking

\- Brute-force attack identification

\- Defensive automation concepts

\- Practical use of security tooling (OpenSSH, Fail2Ban)



---



\## Why This Matters



SSH brute-force attacks remain one of the most common real-world threats against exposed Linux systems.  

These labs demonstrate the \*\*hands-on skills required to detect, investigate, and mitigate these attacks\*\*, rather than relying solely on theoretical knowledge.



This mirrors the workflow used by:

\- SOC Analysts

\- Detection Engineers

\- Blue Team / IR teams



---



\## Next Steps



Future iterations of this section will expand into:

\- Detection tuning and false-positive reduction

\- Alert enrichment

\- SIEM-style correlation

\- Expanded automation using Python



