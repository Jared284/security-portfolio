# AWS Cloud Security Monitoring Lab

## Overview

This lab demonstrates an AWS-native security monitoring pipeline that centralizes, detects, and alerts on security-relevant activity from two sources:

- **Host-level Linux authentication logs** from an EC2 instance
- **AWS control-plane activity** captured by CloudTrail

The project shows how workload telemetry and AWS account activity can be collected, converted into detection signals, and escalated through CloudWatch alarms and SNS notifications.

This lab was designed to be understandable, testable, and defensible in an interview setting. The focus was not only on enabling logging, but on validating the full monitoring path from event generation to alert delivery.

---

## What This Lab Proves

This project demonstrates that I can:

- Forward Linux authentication logs from an EC2 instance into CloudWatch Logs
- Configure CloudTrail to capture AWS control-plane activity
- Deliver CloudTrail logs to both S3 and CloudWatch Logs
- Build CloudWatch metric filters from real security events
- Create custom CloudWatch metrics for detection use cases
- Configure CloudWatch alarms with defined thresholds
- Route alarm notifications through Amazon SNS
- Validate detections end to end using controlled security events
- Explain the difference between host-level monitoring and cloud control-plane monitoring

---

## Architecture Summary

The lab contains two parallel monitoring paths.

### 1. Host-Level Monitoring Path

The EC2 instance generates Linux SSH authentication telemetry in:

```text
/var/log/auth.log
```

The CloudWatch Agent forwards these logs into CloudWatch Logs. A metric filter detects repeated invalid-user SSH login attempts and triggers a CloudWatch alarm when the activity crosses the configured threshold.

### 2. AWS Control-Plane Monitoring Path

CloudTrail records AWS management activity such as:

- Security group ingress rule additions
- Security group ingress rule removals
- IAM managed policy attachments

CloudTrail delivers this telemetry to:

- **S3** for retained log storage
- **CloudWatch Logs** for monitoring and detection

CloudWatch metric filters identify selected CloudTrail events, convert them into custom metrics, and trigger CloudWatch alarms that notify through SNS.

---

## AWS Services Used

- Amazon EC2
- Amazon CloudWatch Logs
- Amazon CloudWatch Metrics
- Amazon CloudWatch Alarms
- Amazon CloudWatch Agent
- AWS CloudTrail
- Amazon S3
- Amazon SNS
- AWS IAM

---

## Lab Architecture

```text
HOST-LEVEL MONITORING PATH

SSH Activity
    ↓
EC2 Ubuntu Instance
    ↓
/var/log/auth.log
    ↓
CloudWatch Agent
    ↓
CloudWatch Logs
    ↓
Metric Filter: Invalid user
    ↓
Custom Metric: InvalidUserSSHAttempts
    ↓
CloudWatch Alarm
    ↓
SNS Email Notification


AWS CONTROL-PLANE MONITORING PATH

AWS Account Activity
(Security Group Changes / IAM Policy Attachment)
    ↓
CloudTrail
    ├── S3 Bucket
    │   └── Retained CloudTrail logs
    │
    └── CloudWatch Logs
        ↓
        Metric Filters
        ↓
        Custom Metrics
        ↓
        CloudWatch Alarms
        ↓
        SNS Email Notifications
```

---

## Project Structure

```text
03-cloud-security-monitoring/
├── 01-architecture/
├── 02-setup-and-build/
├── 03-log-sources-and-ingestion/
├── 04-attack-simulation/
├── 05-detection-engineering/
├── 06-alerting-and-response/
├── 07-reflections-and-improvements/
└── screenshots/
```

---

## Detections Implemented

| Detection | Telemetry Source | Event / Pattern | Detection Output | Alerting |
|---|---|---|---|---|
| Invalid SSH user attempts | `/var/log/auth.log` | `Invalid user` | `InvalidUserSSHAttempts` custom metric | CloudWatch alarm + SNS |
| Security group ingress rule removal | CloudTrail | `RevokeSecurityGroupIngress` | `RevokeSecurityGroupIngressEvents` custom metric | CloudWatch alarm + SNS |
| Security group ingress rule addition | CloudTrail | `AuthorizeSecurityGroupIngress` | `AuthorizeSecurityGroupIngressEvents` custom metric | CloudWatch alarm + SNS |
| IAM managed policy attachment | CloudTrail | `AttachUserPolicy` | `AttachUserPolicyEvents` custom metric | CloudWatch alarm + SNS |

---

## Detection 1: Invalid SSH User Attempts

This detection identifies repeated invalid-user SSH login attempts against the EC2 instance.

### Signal

```text
Invalid user
```

### Log Source

```text
/var/log/auth.log
```

### Detection Logic

CloudWatch Logs metric filter:

```text
"Invalid user"
```

### Custom Metric

```text
Namespace: CloudSecurityMonitoring
Metric: InvalidUserSSHAttempts
```

### Alarm

```text
Alarm: invalid-user-ssh-attempts-alarm
Threshold: Greater than 3 attempts within 5 minutes
```

### Validation

This detection was validated by generating repeated fake SSH login attempts against the EC2 instance, confirming the events in `/var/log/auth.log`, confirming log ingestion into CloudWatch Logs, verifying the custom metric, triggering the CloudWatch alarm, and receiving the SNS email notification.

---

## Detection 2: Security Group Ingress Rule Removal

This detection identifies when a security group ingress rule is removed.

### Signal

```text
RevokeSecurityGroupIngress
```

### Telemetry Source

```text
AWS CloudTrail
```

### Detection Logic

CloudWatch Logs metric filter matching CloudTrail events where:

```text
eventSource = ec2.amazonaws.com
eventName = RevokeSecurityGroupIngress
```

### Custom Metric

```text
Namespace: CloudSecurityMonitoring
Metric: RevokeSecurityGroupIngressEvents
```

### Alarm

```text
Alarm: revoke-security-group-ingress-alarm
Threshold: Greater than or equal to 1 event within 5 minutes
```

### Validation

This detection was validated by making a controlled security group ingress rule removal, confirming the event in CloudTrail Event history, confirming CloudWatch Logs ingestion, verifying the custom metric, triggering the CloudWatch alarm, and receiving the SNS email notification.

---

## Detection 3: Security Group Ingress Rule Addition

This detection identifies when a security group ingress rule is added.

### Signal

```text
AuthorizeSecurityGroupIngress
```

### Telemetry Source

```text
AWS CloudTrail
```

### Detection Logic

CloudWatch Logs metric filter matching CloudTrail events where:

```text
eventSource = ec2.amazonaws.com
eventName = AuthorizeSecurityGroupIngress
```

### Custom Metric

```text
Namespace: CloudSecurityMonitoring
Metric: AuthorizeSecurityGroupIngressEvents
```

### Alarm

```text
Alarm: authorize-security-group-ingress-alarm
Threshold: Greater than or equal to 1 event within 5 minutes
```

### Validation

This detection was validated by making a controlled security group ingress rule addition, confirming the event in CloudTrail Event history, confirming CloudWatch Logs ingestion, verifying the custom metric, triggering the CloudWatch alarm, and receiving the SNS email notification.

---

## Detection 4: IAM Managed Policy Attachment

This detection identifies when an IAM managed policy is attached directly to a user.

### Signal

```text
AttachUserPolicy
```

### Telemetry Source

```text
AWS CloudTrail
```

### Detection Logic

CloudWatch Logs metric filter matching CloudTrail events where:

```text
eventSource = iam.amazonaws.com
eventName = AttachUserPolicy
```

### Custom Metric

```text
Namespace: CloudSecurityMonitoring
Metric: AttachUserPolicyEvents
```

### Alarm

```text
Alarm: attach-user-policy-alarm
Threshold: Greater than or equal to 1 event within 5 minutes
```

### Validation

This detection was validated by attaching the AWS managed `IAMReadOnlyAccess` policy to a test IAM user, confirming the event in CloudTrail Event history, confirming CloudWatch Logs ingestion, verifying the custom metric, triggering the CloudWatch alarm, and receiving the SNS email notification.

---

## Validation Summary

| Monitoring Path | Event Generated | Log Source Confirmed | Metric Created | Alarm Triggered | SNS Delivered |
|---|---:|---:|---:|---:|---:|
| Host-level SSH monitoring | Yes | Yes | Yes | Yes | Yes |
| Security group rule removal | Yes | Yes | Yes | Yes | Yes |
| Security group rule addition | Yes | Yes | Yes | Yes | Yes |
| IAM policy attachment | Yes | Yes | Yes | Yes | Yes |

---

## Screenshots and Evidence

Screenshots are included throughout the lab to support the technical claims made in each section.

Evidence includes:

- EC2 host log generation
- CloudWatch Agent log forwarding
- CloudWatch Logs ingestion
- CloudTrail event history
- CloudTrail delivery to CloudWatch Logs
- Metric filter configuration
- Custom metric datapoints
- CloudWatch alarm state changes
- SNS email alert delivery

The goal of the screenshots is to prove the monitoring pipeline worked end to end, not to include screenshots for their own sake.

---

## Security Concepts Demonstrated

- Cloud security monitoring
- Centralized logging
- Host-level telemetry collection
- AWS control-plane monitoring
- Detection engineering
- CloudWatch metric filters
- Threshold-based alerting
- CloudTrail audit logging
- Incident detection workflow
- Security event validation
- Alert routing through SNS

---

## Why This Lab Matters

Cloud security monitoring requires visibility into more than one layer.

A defender needs to understand what is happening:

- inside the workload
- across the AWS account
- at the network-control-plane level
- at the identity-control-plane level

This lab demonstrates that distinction by combining Linux authentication telemetry with AWS CloudTrail management events.

The result is a small but realistic AWS-native monitoring workflow that shows how raw security events can become actionable alerts.

---

## Final Outcome

The final lab includes four validated monitoring and alerting paths:

1. Repeated invalid-user SSH attempts against an EC2 instance
2. Security group ingress rule removal
3. Security group ingress rule addition
4. IAM managed policy attachment to a user

Each detection was validated through controlled event generation, log review, custom metric creation, alarm triggering, and SNS notification delivery.

This project demonstrates practical cloud security monitoring across host telemetry, AWS network-control-plane activity, and AWS identity-control-plane activity.

---

## Future Improvements

Future improvements could include:

- Failed AWS console login detection
- `CreateAccessKey` detection
- `PutUserPolicy` detection
- `AttachRolePolicy` detection
- CloudTrail disablement or modification detection
- Additional host-side detections beyond SSH activity
- Cross-correlation between host-level and CloudTrail telemetry
- Integration with a SIEM such as Splunk, Elastic, or Microsoft Sentinel

---

## Author

Jared Weiss  
IT & Cybersecurity Student  
GitHub: https://github.com/Jared284
