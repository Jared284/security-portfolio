# Cybersecurity Labs Index

This directory contains hands-on cybersecurity labs focused on cloud security, detection engineering, IAM hardening, API security, and Linux security operations.

Each lab is built to show a complete security workflow: setup, security activity, telemetry, detection or hardening, validation evidence, screenshots, and lessons learned.

[Back to main portfolio README](../README.md)

---

## Lab Overview

| Lab | Focus Area | Primary Tools / Services | What It Proves |
|---|---|---|---|
| [03 - AWS Cloud Security Monitoring](./03-cloud-security-monitoring/) | Cloud Security / Detection Engineering | EC2, CloudTrail, CloudWatch Logs, metric filters, alarms, SNS | Ability to build and validate end-to-end AWS security monitoring and alerting |
| [04 - AWS IAM Least Privilege Hardening](./04-iam-least-privilege-hardening/) | IAM / Access Control | IAM, S3, AWS CLI, CloudTrail | Ability to identify over-permissioned access, enforce least privilege, and validate denied actions |
| [02 - AWS Banking API Security](./02-banking-api-security/) | API Security / Authorization | API Gateway, Lambda, DynamoDB, CloudWatch Logs | Ability to identify and remediate Broken Object Level Authorization in a serverless API |
| [01 - SSH Brute Force Detection](./01-ssh-bruteforce-detection/) | Linux Security / Detection | Ubuntu, OpenSSH, `/var/log/auth.log`, Fail2Ban, Python | Ability to analyze authentication telemetry, detect brute-force behavior, and validate mitigation |

---

## Recommended Review Order

Start with the cloud security labs first:

1. [AWS Cloud Security Monitoring](./03-cloud-security-monitoring/)
2. [AWS IAM Least Privilege Hardening](./04-iam-least-privilege-hardening/)
3. [AWS Banking API Security](./02-banking-api-security/)
4. [SSH Brute Force Detection](./01-ssh-bruteforce-detection/)

Labs 3 and 4 are the strongest cloud-focused projects and should be reviewed first.

---

## Lab Structure

Most labs follow this general structure:

```text
README.md
01-architecture/
02-setup-and-build/
03-operations-and-commands/
04-detection-or-defense/
05-attacks-and-simulation/
06-reflections-and-improvements/
screenshots/
```

Some labs include additional folders when needed, such as:

```text
policies/
planning.md
07-reflections-and-improvements/
```

The structure is intentionally consistent so each project can be reviewed quickly.

---

## Core Security Themes

Across the lab set, the main themes are:

- Cloud security monitoring
- AWS IAM hardening
- Detection engineering
- API authorization
- Linux authentication log analysis
- Least privilege
- Security event validation
- Alerting and response
- Evidence-based technical documentation

---

## Evidence Included

The labs include evidence such as:

- AWS screenshots
- CloudTrail events
- CloudWatch logs
- CloudWatch alarms
- SNS alert validation
- AWS CLI command output
- Linux authentication logs
- API request and response testing
- IAM denied-action validation
- Remediation notes and limitations

The goal is to make each project reviewable through actual evidence, not just written summaries.

---

## Lab Status

| Lab | Status |
|---|---|
| [01 - SSH Brute Force Detection](./01-ssh-bruteforce-detection/) | Complete |
| [02 - AWS Banking API Security](./02-banking-api-security/) | Complete |
| [03 - AWS Cloud Security Monitoring](./03-cloud-security-monitoring/) | Complete |
| [04 - AWS IAM Least Privilege Hardening](./04-iam-least-privilege-hardening/) | Complete |

---

## Notes

These are controlled lab environments, not production systems.

The purpose of the labs is to demonstrate practical security thinking: building systems, generating security activity, collecting evidence, validating controls, documenting limitations, and explaining the results clearly.
