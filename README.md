# Cybersecurity Portfolio

I am Jared Weiss, an IT and cybersecurity student focused on cloud security, detection engineering, IAM hardening, and application security.

This portfolio contains hands-on cybersecurity labs built around practical security workflows: building environments, generating security activity, collecting telemetry, identifying weaknesses, validating controls, and documenting evidence.

The goal of this portfolio is simple: show security work that can be reviewed, tested, and understood through clear documentation, screenshots, logs, commands, and limitations.

---

## Featured Labs

| Lab | Focus Area | What It Demonstrates |
|---|---|---|
| [03 - AWS Cloud Security Monitoring](./labs/03-cloud-security-monitoring/) | Cloud Security / Detection Engineering | Built an AWS monitoring pipeline using EC2 auth logs, CloudTrail, CloudWatch Logs, metric filters, alarms, and SNS notifications |
| [04 - AWS IAM Least Privilege Hardening](./labs/04-iam-least-privilege-hardening/) | IAM / Access Control | Identified over-permissioned AWS IAM access, replaced broad permissions with scoped least-privilege policies, and validated denied actions |
| [02 - AWS Banking API Security](./labs/02-banking-api-security/) | API Security / Authorization | Built a vulnerable serverless banking API, demonstrated a Broken Object Level Authorization issue, and remediated it with backend ownership validation |
| [01 - SSH Brute Force Detection](./labs/01-ssh-bruteforce-detection/) | Linux Security / Detection | Simulated SSH brute-force activity, analyzed Linux authentication logs, configured Fail2Ban, and built custom detection logic |

---

## Recommended Review Path

For recruiters, engineers, or security professionals reviewing this portfolio, start here:

1. [AWS Cloud Security Monitoring](./labs/03-cloud-security-monitoring/)
2. [AWS IAM Least Privilege Hardening](./labs/04-iam-least-privilege-hardening/)
3. [AWS Banking API Security](./labs/02-banking-api-security/)
4. [SSH Brute Force Detection](./labs/01-ssh-bruteforce-detection/)

Labs 3 and 4 are the strongest cloud security projects and should be reviewed first.

For the full lab directory, see the [Cybersecurity Labs Index](./labs/).

---

## Skills Demonstrated

### Cloud Security

- AWS CloudTrail
- Amazon CloudWatch Logs
- CloudWatch metric filters
- CloudWatch alarms
- Amazon SNS alerting
- Amazon EC2
- Amazon S3
- AWS IAM
- AWS Lambda
- Amazon API Gateway
- Amazon DynamoDB

### Detection Engineering

- Linux authentication log analysis
- AWS control-plane monitoring
- Security event generation
- Log-based detection logic
- CloudWatch metric filters
- Alert validation
- Failed-login pattern analysis
- Python-based detection logic

### Identity and Access Management

- IAM policy analysis
- Least privilege policy design
- Permission scoping
- Denied-action validation
- AWS CLI access testing
- CloudTrail audit review
- Privilege escalation prevention

### Application Security

- Broken Object Level Authorization
- Server-side authorization enforcement
- API request testing
- Backend ownership validation
- Secure remediation validation
- Serverless application security

### Security Operations

- Log review
- Alert validation
- Evidence collection
- Incident-style analysis
- Technical documentation
- Limitations and improvement planning

---

## Portfolio Approach

Each lab is built around a complete security workflow:

```text
Build the environment
        ↓
Generate security activity
        ↓
Collect logs and evidence
        ↓
Identify the security issue
        ↓
Implement detection or hardening
        ↓
Validate the result
        ↓
Document what worked, what failed, and what should improve
```

The focus is not just tool usage. The focus is proving security outcomes with evidence.

---

## Current Lab Set

| Lab | Status | Summary |
|---|---|---|
| [01 - SSH Brute Force Detection](./labs/01-ssh-bruteforce-detection/) | Complete | Linux SSH brute-force detection and response lab |
| [02 - AWS Banking API Security](./labs/02-banking-api-security/) | Complete | Serverless API authorization lab focused on BOLA detection and remediation |
| [03 - AWS Cloud Security Monitoring](./labs/03-cloud-security-monitoring/) | Complete | AWS logging, detection, alerting, and monitoring lab |
| [04 - AWS IAM Least Privilege Hardening](./labs/04-iam-least-privilege-hardening/) | Complete | AWS IAM permission hardening and least-privilege validation lab |

---

## Notes

These projects are lab environments, not production systems. Each lab is documented with setup steps, technical decisions, validation evidence, screenshots, and known limitations.

The purpose is to demonstrate practical security thinking: how to build, test, detect, harden, validate, and document security work.

---

## Author

Jared Weiss  
IT and Cybersecurity Student  
GitHub: [Jared284](https://github.com/Jared284)
