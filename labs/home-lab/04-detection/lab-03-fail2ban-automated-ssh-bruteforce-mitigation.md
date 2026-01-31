# Lab 03 – Fail2Ban Automated SSH Brute-Force Mitigation



## Objective

Detect repeated SSH authentication failures and automatically block offending source IPs using Fail2Ban.



## Environment

- Ubuntu Server with OpenSSH enabled

- Fail2Ban installed and running

- Log source: `auth.log` (monitored via systemd backend)

- Attacker source: Windows host on local network



## Attack Simulation

Multiple SSH login attempts were generated from a Windows host using invalid credentials to simulate brute-force behavior.



## Detection Logic

- `sshd` logs repeated authentication failures to `auth.log`

- Fail2Ban monitors SSH logs via the systemd backend

- When failed attempts exceed the configured retry threshold within a defined time window, Fail2Ban triggers enforcement



## Automated Response

- Source IP exceeded the configured retry threshold

- Fail2Ban automatically banned the offending IP

- Firewall rules were updated to block further SSH connections from the source



## Evidence

Fail2Ban confirmed the active SSH jail and ban status:



```bash

sudo fail2ban-client status

sudo fail2ban-client status sshd



