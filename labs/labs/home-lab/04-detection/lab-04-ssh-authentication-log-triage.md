# Lab 04 – SSH Authentication Log Triage



## Objective

Perform manual triage of SSH authentication logs to assess whether observed failures indicate malicious activity or benign user error.



## Environment

- Ubuntu Server with OpenSSH enabled

- Log source: `/var/log/auth.log`

- Fail2Ban active for automated enforcement



## Observed Activity

Multiple failed SSH authentication attempts were observed from a single source IP within a short time window.



## Indicators

- Repeated authentication failures from one IP address

- Usernames targeted were invalid or non-existent

- Failures occurred in rapid succession



## Assessment

The observed pattern is consistent with automated brute-force probing rather than accidental user error. The use of invalid usernames and repeated attempts within a short time frame increases confidence in malicious intent.



## Response

After exceeding the configured retry threshold, the source IP was automatically banned by Fail2Ban. Subsequent SSH connection attempts from the same source failed.



## Commands Used

```bash

grep "Failed password" /var/log/auth.log

sudo fail2ban-client status sshd



