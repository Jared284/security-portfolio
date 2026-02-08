# 02 – Setup and Build

## Objective
Stand up a minimal, isolated Linux server environment suitable for SSH-focused security monitoring and detection labs, while preserving analyst visibility and controlled network access.

This stage establishes the infrastructure foundation used by all downstream detection and attack simulations.

---

## Environment & Tooling

Host OS:
- Windows 11

Hypervisor:
- Oracle VirtualBox

Guest OS:
- Ubuntu Server 24.04.3 LTS (amd64)

Virtual Machine:
- ubuntu-server-lab

Primary Services:
- OpenSSH Server (sshd)
- Fail2Ban

---

## Network Architecture

Host-Only Adapter:
- Purpose: Maintain isolation while allowing SSH access from the Windows host
- Guest IP: 192.168.56.101

NAT Adapter (Temporary):
- Purpose: Enable outbound internet access for package installation
- Used only during setup
- Retained alongside host-only to preserve isolation model

Design Tradeoff:
- Host-only networking alone prevented outbound connectivity (apt update failed)
- NAT was intentionally added instead of bridged networking to avoid LAN exposure

---

## Build Steps (High-Level)

1. Installed Ubuntu Server on VirtualBox
2. Configured Host-Only Adapter for controlled access
3. Verified IP addressing via ip a
4. Installed and enabled OpenSSH server
5. Validated SSH access from Windows PowerShell
6. Temporarily added NAT adapter to enable package installation
7. Installed and enabled Fail2Ban
8. Created a dedicated jail override for SSH monitoring
9. Restarted and validated services

This setup intentionally separates connectivity requirements from security posture.

---

## Commands Used

sudo apt update  
sudo apt install openssh-server  
ip a  
systemctl status ssh  
sudo apt install fail2ban  
sudo systemctl enable --now fail2ban  
sudo nano /etc/fail2ban/jail.d/sshd.local  
sudo systemctl restart fail2ban  

---

## Key Configuration Decisions

- SSH authentication logs are written to /var/log/auth.log for manual triage and automated parsing
- Fail2Ban configured using a jail override rather than modifying default configuration files
- systemd backend selected for Fail2Ban to align with Ubuntu 24.04 logging behavior
- NAT adapter retained only to satisfy package installation requirements, not for ongoing operations

---

## Validation

- Successful SSH authentication from the Windows host confirmed connectivity
- systemctl status ssh verified SSH service health
- systemctl status fail2ban confirmed Fail2Ban service startup
- SSH authentication events confirmed in /var/log/auth.log

---

## Constraints & Limitations

- Host-only networking alone cannot support package installation
- NAT introduces outbound access but is not representative of a fully isolated production server
- Single-host lab does not simulate lateral movement or multi-node visibility

These limitations are intentional and addressed in later stages through detection logic and attack simulation.
