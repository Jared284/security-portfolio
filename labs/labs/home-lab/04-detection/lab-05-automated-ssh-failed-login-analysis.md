\# Lab 05 – Automated SSH Failed Login Analysis



\## Objective

Automate the identification and aggregation of failed SSH authentication attempts to replicate analyst triage performed manually in prior labs.



\## Environment

\- Ubuntu Server with OpenSSH enabled

\- Log source: `/var/log/auth.log`

\- Analysis performed using a custom script

\- Prior manual triage methodology established in Lab 04



\## Method

SSH authentication logs were parsed to extract source IP addresses associated with failed login attempts. Failures were aggregated by IP address and ranked by frequency to highlight suspicious activity.



This automation mirrors analyst triage logic by emphasizing behavior patterns rather than individual log events.



\## Key Result

The script successfully identified repeated SSH authentication failures originating from a single source IP. The detected pattern matched indicators previously observed during manual log analysis.



\## Outcome

Manual SSH log triage from Lab 04 was translated into a repeatable, automated workflow. This reduced reliance on raw log inspection while preserving analyst visibility into suspicious authentication behavior.



\## Defensive Insight

Automating log aggregation enables faster detection and prioritization of brute-force activity while maintaining transparency into underlying evidence. Script-based analysis bridges the gap between raw telemetry and actionable security insights.



