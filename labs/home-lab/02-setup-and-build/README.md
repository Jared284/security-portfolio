# 02 – Environment Setup and Build

## Objective

Build a minimal, isolated Linux server environment capable of generating and monitoring SSH authentication telemetry for security detection experiments.

This environment serves as the infrastructure foundation for all subsequent detection engineering and attack simulation stages.

---

## Environment & Tooling

**Host OS**
- Windows 11

**Hypervisor**
- Oracle VirtualBox

**Guest OS**
- Ubuntu Server 24.04.3 LTS (amd64)

**Virtual Machine**
- ubuntu-server-lab

**Primary Services**
- OpenSSH Server (sshd)
- Fail2Ban

---

## Network Architecture

**Host-Only Adapter**
- Purpose: Maintain isolation while allowing SSH access from the Windows host
- Guest IP: 192.168.56.101

**NAT Adapter (Temporary)**
- Purpose: Enable outbound internet access for package installation
- Used only during setup
- Retained alongside host-only to preserve the isolation model

### Design Tradeoff

Host-only networking alone prevented outbound connectivity, which caused package management operations such as `apt update` to fail.

A NAT adapter was intentionally added instead of bridged networking to avoid exposing the lab environment to the local network.

The objective was to maintain a controlled attack surface while still allowing required package installation during the initial build stage.

---

## Build Process (High-Level)

1. Provisioned Ubuntu Server 24.04 VM within VirtualBox  
2. Configured Host-Only Adapter to allow SSH access only from the Windows host  
3. Verified network configuration using `ip a`  
4. Installed and enabled OpenSSH Server  
5. Validated SSH connectivity from Windows PowerShell  
6. Temporarily added NAT adapter to allow outbound package installation  
7. Installed and enabled Fail2Ban  
8. Created a dedicated jail override for SSH monitoring  
9. Restarted and validated services  

This setup intentionally separates connectivity requirements from the security posture of the environment.

---

## Commands Used

```
sudo apt update
sudo apt install openssh-server

ip a
systemctl status ssh

sudo apt install fail2ban
sudo systemctl enable --now fail2ban

sudo nano /etc/fail2ban/jail.d/sshd.local

sudo systemctl restart fail2ban
```

---

## Key Configuration Decisions

- SSH authentication logs are written to `/var/log/auth.log` for manual triage and automated parsing
- Fail2Ban is configured using a jail override instead of modifying default configuration files
- The `systemd` backend is used for Fail2Ban to align with Ubuntu 24.04 logging behavior
- A NAT adapter is used only to support package installation during setup

These decisions preserve maintainability and ensure the lab environment mirrors realistic Linux security configurations.

---

## Validation

- Successful SSH authentication from the Windows host confirmed SSH connectivity
- SSH access validated on TCP port 22 from the Windows host
- `systemctl status ssh` verified SSH service health
- `systemctl status fail2ban` confirmed Fail2Ban service startup
- SSH authentication events verified in `/var/log/auth.log`

---

## Constraints & Limitations

- Host-only networking alone cannot support package installation
- NAT introduces outbound access but is not representative of a fully isolated production server
- The lab currently operates as a single-host environment and does not simulate lateral movement or multi-node visibility

These constraints are intentional and will be addressed in later stages through detection engineering and controlled attack simulation.
