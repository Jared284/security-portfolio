# 01 - Architecture

## Purpose

This section explains the architecture of the AWS Cloud Security Monitoring Lab.

The lab centralizes security-relevant telemetry from two layers:

- **Host-level activity** from an EC2 Linux instance
- **AWS control-plane activity** from CloudTrail

The architecture is designed to collect logs, apply detection logic, generate custom metrics, trigger alarms, and deliver alerts through SNS.

The goal is to demonstrate a small but realistic AWS-native monitoring workflow that combines workload visibility with cloud account visibility.

---

## Architecture Goals

This architecture supports:

- Collection of Linux authentication logs from an EC2 instance
- Collection of AWS control-plane events through CloudTrail
- Centralized logging in CloudWatch Logs
- Long-term CloudTrail log storage in S3
- Detection logic using CloudWatch Logs metric filters
- Custom CloudWatch metrics for security events
- Threshold-based alerting with CloudWatch alarms
- Email notification delivery through SNS
- End-to-end validation from event generation to alert delivery

---

## Core Components

### 1. EC2 Instance

An Ubuntu EC2 instance acts as the monitored host.

This instance generates host-level authentication telemetry when SSH login attempts occur. In this lab, the host is used to validate detection of repeated invalid-user SSH login attempts.

Primary host log source:

```text
/var/log/auth.log
```

---

### 2. CloudWatch Agent

The CloudWatch Agent runs on the EC2 instance and forwards Linux authentication logs to CloudWatch Logs.

This allows host-level security telemetry to be centralized and monitored outside of the instance itself.

---

### 3. CloudWatch Logs

CloudWatch Logs centralizes both host-level logs and CloudTrail-delivered AWS activity logs.

The lab uses two primary log groups:

```text
cloud-security-monitoring-auth
cloud-security-monitoring-cloudtrail
```

The first log group stores Linux authentication logs from the EC2 instance.

The second log group stores AWS control-plane events delivered by CloudTrail.

---

### 4. CloudTrail

CloudTrail records AWS account activity and management events.

This provides visibility into control-plane actions such as:

- Security group ingress rule additions
- Security group ingress rule removals
- IAM managed policy attachments

These events do not appear in the operating-system logs of the EC2 instance, so CloudTrail is required for cloud-side monitoring.

---

### 5. Amazon S3

S3 stores retained CloudTrail logs.

CloudTrail sends logs to S3 for longer-term storage and later review, while CloudWatch Logs supports monitoring, metric filters, and alerting.

---

### 6. Detection Layer

Detection logic is implemented with CloudWatch Logs metric filters.

Metric filters match selected log patterns or CloudTrail events and convert them into custom CloudWatch metrics.

Those metrics become the signals evaluated by CloudWatch alarms.

---

### 7. Alerting Layer

CloudWatch alarms evaluate custom metrics against defined thresholds.

When an alarm enters the `ALARM` state, Amazon SNS sends an email notification.

This completes the detection-to-alerting path.

---

## Monitoring Paths

The lab contains two parallel monitoring paths.

---

## Path 1: Host-Level Monitoring

This path monitors activity occurring inside the EC2 instance.

### Flow

1. SSH activity occurs against the EC2 instance.
2. The instance writes authentication events to `/var/log/auth.log`.
3. The CloudWatch Agent forwards those logs to CloudWatch Logs.
4. A CloudWatch Logs metric filter matches invalid-user SSH activity.
5. The filter publishes datapoints to a custom CloudWatch metric.
6. A CloudWatch alarm evaluates the metric against its threshold.
7. SNS sends an email notification when the alarm triggers.

### Detection Supported

```text
Repeated invalid-user SSH login attempts
```

### Security Value

This path shows how host telemetry can be collected and converted into an actionable alert.

---

## Path 2: AWS Control-Plane Monitoring

This path monitors activity occurring in the AWS account itself.

### Flow

1. Security-relevant AWS activity occurs in the account.
2. CloudTrail records the management event.
3. CloudTrail delivers logs to S3 and CloudWatch Logs.
4. CloudWatch Logs metric filters match selected CloudTrail events.
5. The filters publish datapoints to custom CloudWatch metrics.
6. CloudWatch alarms evaluate the metrics against their thresholds.
7. SNS sends email notifications when alarms trigger.

### Detections Supported

```text
Security group ingress rule removal
Security group ingress rule addition
IAM managed policy attachment to a user
```

### Security Value

This path shows how AWS account activity can be monitored beyond the operating system of a single host.

It provides visibility into network-control-plane and identity-control-plane activity.

---

## Implemented Detection Use Cases

| Detection | Source | Event / Pattern | Metric | Alarm |
|---|---|---|---|---|
| Invalid SSH user attempts | EC2 `/var/log/auth.log` | `Invalid user` | `InvalidUserSSHAttempts` | `invalid-user-ssh-attempts-alarm` |
| Security group ingress rule removal | CloudTrail | `RevokeSecurityGroupIngress` | `RevokeSecurityGroupIngressEvents` | `revoke-security-group-ingress-alarm` |
| Security group ingress rule addition | CloudTrail | `AuthorizeSecurityGroupIngress` | `AuthorizeSecurityGroupIngressEvents` | `authorize-security-group-ingress-alarm` |
| IAM managed policy attachment | CloudTrail | `AttachUserPolicy` | `AttachUserPolicyEvents` | `attach-user-policy-alarm` |

All detections publish custom metrics under the following namespace:

```text
CloudSecurityMonitoring
```

---

## Architecture Diagram

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
cloud-security-monitoring-auth
        ↓
Metric Filter
invalid-user-ssh-attempts
        ↓
Custom Metric
CloudSecurityMonitoring / InvalidUserSSHAttempts
        ↓
CloudWatch Alarm
invalid-user-ssh-attempts-alarm
        ↓
SNS Notification


AWS CONTROL-PLANE MONITORING PATH

AWS Account Activity
(Security Group Changes / IAM Policy Attachment)
        ↓
CloudTrail
        ├──→ S3 Bucket
        │     Retained CloudTrail Log Storage
        │
        └──→ CloudWatch Logs
              cloud-security-monitoring-cloudtrail
                    ↓
              Metric Filters
              - revoke-security-group-ingress
              - authorize-security-group-ingress
              - attach-user-policy
                    ↓
              Custom Metrics
              - RevokeSecurityGroupIngressEvents
              - AuthorizeSecurityGroupIngressEvents
              - AttachUserPolicyEvents
                    ↓
              CloudWatch Alarms
              - revoke-security-group-ingress-alarm
              - authorize-security-group-ingress-alarm
              - attach-user-policy-alarm
                    ↓
              SNS Notification
```

---

## Why This Architecture Matters

This architecture is intentionally broader than a single host-based detection setup.

A host-only monitoring lab can show how one system is attacked and how local logs can be analyzed. This lab goes further by showing how a defender can monitor:

- Activity inside a Linux system
- Activity across the AWS account control plane
- Network-control-plane changes through security group events
- Identity-control-plane changes through IAM events

That distinction matters because real cloud security monitoring depends on visibility across multiple telemetry sources.

---

## Final Architecture Summary

This lab combines host monitoring and cloud control-plane monitoring into one AWS-native security monitoring workflow.

The final architecture includes:

- One EC2 instance as the monitored system
- CloudWatch Agent for host log forwarding
- CloudWatch Logs for centralized log collection
- CloudTrail for AWS account activity logging
- S3 for retained CloudTrail storage
- CloudWatch metric filters for detection logic
- CloudWatch alarms for threshold-based alerting
- SNS for notification delivery

Together, these components create a basic but realistic cloud security monitoring pipeline.
