\# Lab Architecture



This document describes the system design, trust boundaries, and security controls for the Linux home lab used to simulate SOC-style detection and response workflows.



---



\## Host Environment



\- \*\*Host OS:\*\* Windows 11  

\- \*\*Virtualization Platform:\*\* Oracle VirtualBox



---



\## Guest Environment



\- \*\*Guest OS:\*\* Ubuntu Server 24.04 LTS  

\- \*\*Role:\*\* Central log source and detection host



---



\## Network Architecture



\- \*\*Network Mode:\*\* Host-only networking  

\- \*\*Guest IP Address:\*\* `192.168.56.101`  

\- \*\*Exposed Services:\*\*

&nbsp; - SSH (`tcp/22`) accessible \*\*only from the host system\*\*



---



\## Trust Boundaries



\- The guest system is fully isolated from the external internet  

\- Only the host machine is permitted to initiate SSH connections  

\- No inbound access from untrusted networks is allowed  



This isolation ensures all observed authentication activity is either:

\- Legitimate administrative access, or

\- Controlled attack simulation originating from the host



---



\## Security Controls



\- `fail2ban` deployed on the Ubuntu Server

\- `sshd` jail enabled and monitoring authentication logs

\- Threshold-based IP banning enforced automatically

\- Firewall rules dynamically updated to block offending source IPs



---



\## Threat Model



This lab is designed to detect and respond to:



\- SSH brute-force authentication attempts

\- Repeated failed login activity from a single source IP

\- Automated credential probing against non-existent users



The architecture intentionally limits attack surface to focus on \*\*detection logic and response behavior\*\*, rather than perimeter defense complexity.



