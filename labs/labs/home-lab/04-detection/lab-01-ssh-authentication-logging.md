\# Lab 01 – SSH Authentication Logging



\## Objective

Observe how SSH authentication attempts are logged on an Ubuntu Server and identify indicators of legitimate versus suspicious access.



\## Environment

\- OS: Ubuntu Server LTS

\- Service: OpenSSH

\- Client: Windows host (SSH client)

\- Log file: `/var/log/auth.log`



\## Procedure

\- Enabled the OpenSSH server on Ubuntu

\- Connected to the server via SSH from a Windows host

\- Generated both successful and failed authentication attempts



\## Evidence

\- Successful login events recorded with:

&nbsp; - `Accepted password`

\- Failed authentication attempts recorded with:

&nbsp; - `Failed password for invalid user`



\## Analysis

Authentication logs provide defenders with visibility into access attempts.  

Repeated failed logins, especially for invalid users, can indicate brute-force or credential-stuffing activity, while successful logins can be correlated with expected users and source IPs.



\## Outcome

This lab demonstrated how SSH authentication activity is recorded and how logs can be used to distinguish legitimate access from suspicious behavior.



