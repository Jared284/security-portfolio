# Lab 01 – SSH Authentication Logging

## Objective

Observe how SSH authentication attempts are recorded on an Ubuntu server and identify log indicators that differentiate legitimate access from suspicious activity.

This lab establishes the **baseline telemetry** used in later detection experiments.

---

## Environment

- **Server OS:** Ubuntu Server 24.04 LTS  
- **Service:** OpenSSH Server (sshd)  
- **Client:** Windows 11 host (SSH client)  
- **Log Source:** `/var/log/auth.log`  

---

## Procedure

1. Installed and enabled the OpenSSH server on the Ubuntu VM  
2. Initiated SSH connections from the Windows host  
3. Generated both successful and failed login attempts  
4. Observed authentication events written to `/var/log/auth.log`  

Authentication events were inspected using:

```
sudo cat /var/log/auth.log
```

---

## Evidence

### Successful Authentication

Example log entry:

```
Accepted password for analyst from 192.168.56.1 port 53412 ssh2
```

Key fields observed:

- **Username** attempting authentication
- **Source IP address**
- **Timestamp**
- **Authentication result**

---

### Failed Authentication

Example log entry:

```
Failed password for invalid user testuser from 192.168.56.1 port 53413 ssh2
```

Indicators observed:

- Attempts against **invalid usernames**
- Repeated failed password attempts
- Source IP address generating the request

---

## Detection Signal

Important authentication indicators include:

- `Accepted password` → successful login  
- `Failed password` → incorrect password for valid user  
- `Failed password for invalid user` → probing for non-existent accounts  

Repeated failures from the same source IP may indicate:

- brute-force attempts
- credential guessing
- automated login probes

---

## Analysis

SSH authentication logs provide defenders with direct visibility into access attempts against a system.

Even unsuccessful login attempts generate valuable telemetry that can be used to detect suspicious behavior.

Key insights from this lab:

- Authentication logs capture both successful and failed access attempts
- Invalid user attempts are a strong indicator of malicious probing
- Source IP and timestamp fields allow analysts to correlate repeated behavior

These signals become the foundation for brute-force detection and automated defense mechanisms explored in later labs.

---

## Outcome

This lab established a baseline understanding of SSH authentication telemetry.

By examining authentication logs directly, it becomes possible to identify login behavior, investigate suspicious activity, and build detection logic based on authentication patterns.
