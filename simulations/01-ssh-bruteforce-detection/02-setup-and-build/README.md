# 02 – Environment Setup and Build

## Objective

Build a minimal, isolated Linux server environment capable of generating, storing, and analyzing SSH authentication telemetry for detection engineering.

This setup provides the technical foundation for the rest of the lab:

~~~text
Ubuntu Server VM
        ↓
OpenSSH authentication activity
        ↓
/var/log/auth.log
        ↓
Manual investigation
        ↓
Fail2Ban automated response
        ↓
Shell and Python detection logic
~~~

The goal is to create a safe local environment where SSH attacks can be simulated without exposing a real system to the public internet.

---

## Environment Summary

| Component | Configuration |
|---|---|
| Host OS | Windows 11 |
| Hypervisor | Oracle VirtualBox |
| Guest OS | Ubuntu Server 24.04.3 LTS |
| VM Name | `ubuntu-server-lab` |
| Target IP | `192.168.56.101` |
| Source IP | `192.168.56.1` |
| Primary Service | OpenSSH Server |
| Defensive Tool | Fail2Ban |
| Primary Log Source | `/var/log/auth.log` |

Evidence:

![VM IP Address](../screenshots/01-vm-ip-address.png)

---

## Build Goals

The environment was built to support five core functions:

1. Generate real SSH authentication events
2. Store authentication activity in Linux system logs
3. Validate SSH service availability
4. Automatically block repeated failed login attempts with Fail2Ban
5. Support custom detection logic using shell commands and Python scripts

This setup intentionally favors simplicity, visibility, and repeatability.

---

## Virtual Machine Setup

An Ubuntu Server VM was created in Oracle VirtualBox.

Core VM role:

~~~text
Ubuntu Server VM = monitored Linux host + detection target
~~~

The VM was configured to act as the SSH target for all controlled testing.

The Windows host was used as:

- the administrative workstation
- the SSH client
- the controlled attack simulation source

This created a simple but realistic attacker-to-server telemetry flow.

---

## Network Configuration

The lab uses host-only networking for safe SSH testing.

| Adapter Type | Purpose |
|---|---|
| Host-only Adapter | Allows Windows host to SSH into Ubuntu VM without public exposure |
| NAT Adapter | Used during setup for outbound package installation and updates |

The Ubuntu VM was assigned the lab IP:

~~~text
192.168.56.101
~~~

The Windows host used the host-only adapter IP:

~~~text
192.168.56.1
~~~

This allowed controlled SSH activity from the host to the VM while keeping the target system isolated from the public internet.

---

## Network Design Tradeoff

Host-only networking is safer because it prevents public exposure, but it does not provide outbound internet access by default.

A NAT adapter was used during setup so the VM could perform package installation and updates.

This was needed for:

- `apt update`
- OpenSSH installation
- Fail2Ban installation
- system package dependencies

NAT was used instead of bridged networking because bridged mode could expose the VM more broadly on the local network.

The design goal was:

~~~text
Allow required setup connectivity without turning the VM into a publicly reachable target.
~~~

---

## OpenSSH Installation

OpenSSH Server was installed to provide the authentication surface for the lab.

Commands used:

~~~bash
sudo apt update
sudo apt install openssh-server
~~~

The SSH service was validated with:

~~~bash
sudo systemctl status ssh
~~~

Expected result:

~~~text
active (running)
~~~

Evidence:

![SSH Service Status](../screenshots/02-ssh-service-status.png)

---

## SSH Connectivity Validation

SSH connectivity was tested from the Windows host using PowerShell.

Example command:

~~~powershell
ssh jared@192.168.56.101
~~~

This confirmed that:

- the Ubuntu VM was reachable from the Windows host
- SSH was listening on TCP/22
- valid credentials could authenticate successfully
- OpenSSH generated authentication events in `/var/log/auth.log`

Evidence:

![Successful SSH Login After Unban](../screenshots/11-successful-ssh-login-after-unban.png)

---

## Fail2Ban Installation

Fail2Ban was installed to provide automated defensive response against repeated SSH authentication failures.

Commands used:

~~~bash
sudo apt install fail2ban
sudo systemctl enable --now fail2ban
sudo systemctl status fail2ban
~~~

Expected result:

~~~text
active (running)
~~~

Evidence:

![Fail2Ban Service Status](../screenshots/03-fail2ban-service-status.png)

---

## SSH Jail Configuration

A local SSH jail override was created for Fail2Ban.

Configuration file:

~~~text
/etc/fail2ban/jail.d/sshd.local
~~~

The purpose of this file is to define SSH-specific monitoring behavior without modifying Fail2Ban's default configuration files.

Core configuration values used during the lab:

| Setting | Value | Purpose |
|---|---|---|
| `enabled` | `true` | Enables SSH monitoring |
| `port` | `ssh` | Monitors SSH service activity |
| `filter` | `sshd` | Uses Fail2Ban's SSH detection filter |
| `logpath` | `/var/log/auth.log` | Watches Linux authentication logs |
| `backend` | `systemd` | Supports Ubuntu 24.04 logging behavior |
| `maxretry` | `5` | Bans after 5 failed attempts |
| `findtime` | `600` | 10-minute detection window |
| `bantime` | `600` | 10-minute ban duration |

After configuration changes, Fail2Ban was restarted:

~~~bash
sudo systemctl restart fail2ban
~~~

The jail was validated with:

~~~bash
sudo fail2ban-client status
sudo fail2ban-client status sshd
~~~

Evidence:

![Fail2Ban SSHD Status Before Attack](../screenshots/04-fail2ban-sshd-status-before-attack.png)

---

## Authentication Log Source

The primary telemetry source for this lab is:

~~~text
/var/log/auth.log
~~~

This file records SSH authentication activity including:

- invalid username probes
- failed password attempts
- accepted password logins
- source IP addresses
- target usernames
- timestamps
- SSH process identifiers

This log source is the foundation for all later detection engineering work in the lab.

---

## Initial Log Validation

After SSH was enabled, authentication activity was validated through direct log review.

Example commands:

~~~bash
sudo grep "sshd" /var/log/auth.log | tail -20
sudo grep "Failed password" /var/log/auth.log | tail -20
sudo grep "Invalid user" /var/log/auth.log | tail -20
sudo grep "Accepted password" /var/log/auth.log | tail -20
~~~

This confirmed that OpenSSH was writing usable telemetry for both failed and successful authentication events.

Evidence:

![Auth Log Failed Password Events](../screenshots/06-auth-log-failed-password-events.png)

![Auth Log Invalid User Events](../screenshots/07-auth-log-invalid-user-events.png)

---

## Build Process Summary

The setup process followed this sequence:

1. Created Ubuntu Server VM in VirtualBox
2. Configured host-only networking
3. Verified VM IP address
4. Installed and enabled OpenSSH Server
5. Validated SSH service health
6. Tested SSH access from Windows PowerShell
7. Added temporary NAT access for package installation
8. Installed and enabled Fail2Ban
9. Created SSH jail override
10. Restarted and validated Fail2Ban
11. Confirmed `/var/log/auth.log` telemetry
12. Generated controlled SSH authentication events
13. Verified defensive response and detection readiness

---

## Commands Used

Core setup commands:

~~~bash
sudo apt update
sudo apt install openssh-server
sudo systemctl status ssh

ip a

sudo apt install fail2ban
sudo systemctl enable --now fail2ban
sudo systemctl status fail2ban

sudo nano /etc/fail2ban/jail.d/sshd.local

sudo systemctl restart fail2ban
sudo fail2ban-client status
sudo fail2ban-client status sshd
~~~

SSH validation from Windows PowerShell:

~~~powershell
ssh jared@192.168.56.101
~~~

Log validation commands:

~~~bash
sudo grep "sshd" /var/log/auth.log | tail -20
sudo grep "Failed password" /var/log/auth.log | tail -20
sudo grep "Invalid user" /var/log/auth.log | tail -20
sudo grep "Accepted password" /var/log/auth.log | tail -20
~~~

---

## Validation Checklist

| Validation Step | Result |
|---|---|
| Ubuntu VM created | Completed |
| Host-only IP assigned | Completed |
| SSH service running | Completed |
| SSH login from Windows host works | Completed |
| Fail2Ban installed and running | Completed |
| SSH jail configured | Completed |
| `/var/log/auth.log` confirmed | Completed |
| Failed login events generated | Completed |
| Invalid user events generated | Completed |
| Successful login events generated | Completed |
| Fail2Ban ban behavior validated | Completed |
| Python detection workflow supported | Completed |

---

## Key Configuration Decisions

| Decision | Reason |
|---|---|
| Ubuntu Server 24.04 LTS | Realistic Linux server environment |
| VirtualBox | Free, local, repeatable lab platform |
| Host-only networking | Prevents public exposure |
| Temporary NAT access | Allows package installation |
| OpenSSH Server | Generates authentication telemetry |
| Fail2Ban | Demonstrates automated response |
| `/var/log/auth.log` | Primary SSH authentication log source |
| Local jail override | Avoids editing default Fail2Ban configs |
| `systemd` backend | Aligns with Ubuntu 24.04 behavior |
| Screenshots | Provides proof of implementation and validation |

---

## Constraints and Limitations

This build is intentionally scoped as a local single-host lab.

Limitations include:

- one Ubuntu target host
- one Windows source host
- one primary source IP
- no centralized SIEM
- no cloud logging pipeline
- no multi-host correlation
- no real internet-facing exposure
- no production firewall or EDR integration

These constraints are acceptable because the objective is to build and validate host-based SSH detection logic in a controlled environment.

---

## Outcome

The environment was successfully built and validated.

The completed setup supports:

- controlled SSH attack simulation
- Linux authentication telemetry generation
- Fail2Ban automated response
- manual log triage
- failed login aggregation
- Python-based detection logic
- failed-to-successful login correlation

This setup provides the foundation for the detection engineering labs that follow.
