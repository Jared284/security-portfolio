\# Lab 02 – SSH Brute-Force Pattern Identification



\## Objective

Identify patterns of repeated SSH authentication failures that may indicate brute-force attack behavior.



\## Environment

\- Ubuntu Server with OpenSSH enabled

\- Log source: `/var/log/auth.log`

\- Attacker source: Windows host on local network



\## Data / Evidence

Analysis of `auth.log` revealed multiple failed SSH authentication attempts originating from a single IP address.



Example log entry:



A total of \*\*6 failed authentication attempts\*\* were observed from:

\- Source IP: `192.168.56.101`



\## Detection Logic

\- Filter SSH authentication failures from `auth.log`

\- Group events by source IP address

\- Identify IPs generating repeated failures over a short time window



Repeated authentication failures from a single source are more indicative of malicious behavior than isolated login errors.



\## Findings

\- The IP address `192.168.56.101` generated multiple failed login attempts

\- Failures were clustered rather than isolated

\- Activity is consistent with early-stage SSH brute-force behavior



\## Commands Used

```bash

grep "Failed password" /var/log/auth.log



