# Lab Architecture

---

## Overview

This document describes the architecture of a Linux-based detection engineering lab designed to simulate SOC-style monitoring, brute-force detection, and automated response workflows.

The environment focuses on **SSH authentication telemetry**, allowing controlled attack simulation and defensive detection development.

The lab models a simplified security operations workflow: log generation, detection logic, and automated response.

---

## System Architecture

```
Windows 11 Host
(Admin + Attack Simulation)
        |
        | SSH Attempts
        v
Ubuntu Server 24.04 VM
        |
        | Authentication Logs (/var/log/auth.log)
        v
fail2ban
        |
        v
Automated Firewall Ban
```

## Host Environment

- **Host OS:** Windows 11  
- **Virtualization Platform:** Oracle VirtualBox  

The host machine acts as the **administrative control point** and the **attack simulation source**.

---

## Guest Environment

- **Guest OS:** Ubuntu Server 24.04 LTS  
- **Role:** Log generation, detection engine, and automated response system  

The Ubuntu server functions as the **central log source and detection host**, monitoring authentication activity and enforcing automated security responses.

---

## Network Architecture

- **Network Mode:** Host-only networking  
- **Guest IP Address:** 192.168.56.101  

### Exposed Services

- SSH (TCP/22) — accessible only from the host machine

This configuration restricts network access to the local virtualization host and prevents exposure to external networks.

---

## Trust Boundaries

The guest system is fully isolated from external networks.

Only the Windows host is permitted to initiate SSH connections.

All authentication activity observed in the lab environment originates from either:

- legitimate administrative access  
- controlled attack simulation from the host system  

This isolation allows detection logic to be tested without interference from external traffic.

---

## Security Controls

fail2ban is deployed on the Ubuntu server to monitor authentication activity and enforce automated mitigation.

Security mechanisms include:

- sshd jail monitoring authentication logs  
- threshold-based banning of repeated failed login attempts  
- automated firewall rule updates to block offending IP addresses  

---

## Threat Model

This lab simulates common authentication attacks against SSH services.

Detection logic focuses on identifying:

- SSH brute-force authentication attempts  
- repeated failed login attempts from a single source IP  
- automated credential probing against non-existent users  

The architecture intentionally limits attack surface to emphasize **detection and response behavior** rather than perimeter defense.

---

## Deployment Considerations

During initial setup, outbound internet connectivity was required for package installation.

A temporary NAT adapter was added to allow `apt` operations and system updates.

Once dependencies were installed, the environment returned to a host-only networking configuration to maintain isolation.

---

## Key Design Lessons

- Detection logic depends on reliable network connectivity  
- Security controls cannot operate if attack traffic never reaches the monitored system  
- Isolation must be balanced with operational requirements such as package installation and telemetry collection  
