# Lab 03 – Fail2Ban Automated SSH Brute-Force Mitigation

## Objective

Detect repeated SSH authentication failures and automatically block offending source IP addresses using Fail2Ban.

This lab demonstrates how log-based detection signals can trigger automated defensive controls.

---

## Environment

- **Server OS:** Ubuntu Server 24.04 LTS  
- **Service:** OpenSSH Server (sshd)  
- **Defense Tool:** Fail2Ban  
- **Log Source:** `/var/log/auth.log` (monitored using the `systemd` backend)  
- **Attack Source:** Windows host on the local network  

---

## Attack Simulation

Multiple SSH login attempts were generated from the Windows host using invalid credentials to simulate brute-force authentication behavior.

Example command used from the Windows host:

```
ssh invaliduser@192.168.56.101
```

Repeated authentication failures were intentionally triggered to exceed the Fail2Ban retry threshold.

---

## Detection Logic

Fail2Ban monitors authentication logs and detects suspicious behavior using the following logic:

1. `sshd` records authentication failures in `/var/log/auth.log`
2. Fail2Ban continuously monitors these logs
3. If failed login attempts exceed the configured threshold within a defined time window, the source IP is flagged as malicious
4. Fail2Ban automatically bans the offending IP address

This transforms raw log signals into automated defensive action.

---

## Automated Response

Once the threshold was exceeded:

- Fail2Ban added the offending IP to the SSH jail ban list
- Firewall rules were dynamically updated
- Additional SSH authentication attempts from the source IP were blocked

This prevents further brute-force attempts from the same attacker.

---

## Evidence

Fail2Ban service and jail status were verified using:

```
sudo fail2ban-client status
```

Example output:

```
Status
|- Number of jail: 1
`- Jail list: sshd
```

Inspecting the SSH jail:

```
sudo fail2ban-client status sshd
```

Example output:

```
Status for the jail: sshd
|- Filter
|  |- Currently failed: 6
|  |- Total failed: 6
|  `- File list: /var/log/auth.log
`- Actions
   |- Currently banned: 1
   |- Total banned: 1
   `- Banned IP list: 192.168.56.101
```

This confirms that the repeated authentication failures triggered automated enforcement.

---

## Analysis

Fail2Ban acts as a bridge between **detection and response**.

Rather than requiring analysts to manually investigate repeated login failures, Fail2Ban automatically enforces blocking rules when predefined thresholds are exceeded.

Key insights from this lab:

- Log-based detection enables automated security responses
- Brute-force attacks can be mitigated quickly using threshold-based controls
- Automation reduces analyst workload during repeated attack attempts

---

## Outcome

This lab demonstrated how authentication telemetry can trigger automated defensive controls.

By integrating Fail2Ban with SSH authentication logs, the system can detect and block brute-force attack attempts without manual intervention.
