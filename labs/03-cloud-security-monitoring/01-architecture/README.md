# 01 - Architecture

## Purpose

This section defines the architecture for the cloud security monitoring lab.

The goal of this architecture is to centralize security-relevant telemetry from both the operating system and the AWS account, apply detection logic to that telemetry, and generate alerts when suspicious activity occurs.

This lab demonstrates a small but realistic AWS-native monitoring workflow across host-level logs and AWS control-plane activity.

---

## Architecture Goals

The architecture supports the following:

- collection of host-level authentication logs from an EC2 instance
- collection of AWS control-plane logs through CloudTrail
- centralized visibility into multiple telemetry sources
- detection of suspicious behavior using AWS-native services
- alerting when defined detection thresholds are met
- storage of CloudTrail log data for later review
- validation of the full path from event generation to notification delivery

---

## Core Components

### 1. EC2 Instance

A Linux EC2 instance acts as the monitored host.

This instance generates host-level telemetry, including SSH authentication activity. In this lab, the EC2 instance is used to validate detection of repeated invalid-user SSH login attempts.

### 2. CloudWatch Agent

The CloudWatch Agent runs on the EC2 instance and forwards Linux authentication logs into CloudWatch Logs.

The monitored host log file is:

```text
/var/log/auth.log
```

This allows host-level authentication telemetry to be centralized and used for detection.

### 3. CloudWatch Logs

CloudWatch Logs centralizes both host-level logs and CloudTrail-delivered AWS activity logs.

This lab uses two primary log groups:

```text
cloud-security-monitoring-auth
cloud-security-monitoring-cloudtrail
```

The first log group stores Linux authentication logs from the EC2 instance. The second stores AWS control-plane events delivered by CloudTrail.

### 4. CloudTrail

CloudTrail records AWS account activity and management events.

This provides visibility into control-plane actions such as EC2 security group changes and IAM policy attachment activity. These events would not appear in the operating-system logs of the EC2 instance, so CloudTrail is required for cloud-side detection.

### 5. Amazon S3

S3 stores retained CloudTrail logs.

CloudTrail delivers logs to S3 for longer-term storage and later review, while CloudWatch Logs supports near-real-time monitoring and detection.

### 6. Detection Layer

Detection logic is implemented with CloudWatch Logs metric filters.

Metric filters convert selected log patterns and CloudTrail events into custom CloudWatch metrics. Those metrics are then evaluated by CloudWatch alarms.

### 7. Alerting Layer

CloudWatch alarms evaluate custom metrics against thresholds.

Amazon SNS delivers notifications when an alarm enters the `ALARM` state.

---

## Data Flow

The lab contains two main monitoring paths: one host-side path and one cloud-side path.

### Host-Level Monitoring Flow

1. Test SSH activity occurs against the EC2 instance
2. The EC2 instance writes authentication events to `/var/log/auth.log`
3. The CloudWatch Agent forwards those logs to CloudWatch Logs
4. A CloudWatch Logs metric filter matches invalid-user SSH activity
5. A custom CloudWatch metric receives a datapoint
6. A CloudWatch alarm evaluates the metric against its threshold
7. SNS sends an email notification when the alarm triggers

### AWS Control-Plane Monitoring Flow

1. Security-relevant activity occurs in the AWS account
2. CloudTrail records the management event
3. CloudTrail delivers logs to S3 and CloudWatch Logs
4. CloudWatch Logs metric filters match selected CloudTrail events
5. Custom CloudWatch metrics receive datapoints
6. CloudWatch alarms evaluate those metrics against thresholds
7. SNS sends email notifications when alarms trigger

---

## Implemented Detection Use Cases

The architecture supports four validated detections.

### Detection 1 - Invalid SSH User Attempts

This detection identifies repeated invalid-user SSH login attempts against the EC2 instance.

- **Source:** EC2 `/var/log/auth.log`
- **Metric filter:** `invalid-user-ssh-attempts`
- **Metric:** `CloudSecurityMonitoring / InvalidUserSSHAttempts`
- **Alarm:** `invalid-user-ssh-attempts-alarm`

### Detection 2 - Security Group Ingress Rule Removal

This detection identifies security group ingress rule removal activity.

- **Source:** CloudTrail
- **Event name:** `RevokeSecurityGroupIngress`
- **Metric filter:** `revoke-security-group-ingress`
- **Metric:** `CloudSecurityMonitoring / RevokeSecurityGroupIngressEvents`
- **Alarm:** `revoke-security-group-ingress-alarm`

### Detection 3 - Security Group Ingress Rule Addition

This detection identifies security group ingress rule addition activity.

- **Source:** CloudTrail
- **Event name:** `AuthorizeSecurityGroupIngress`
- **Metric filter:** `authorize-security-group-ingress`
- **Metric:** `CloudSecurityMonitoring / AuthorizeSecurityGroupIngressEvents`
- **Alarm:** `authorize-security-group-ingress-alarm`

### Detection 4 - IAM Managed Policy Attachment

This detection identifies IAM managed policies being attached directly to users.

- **Source:** CloudTrail
- **Event name:** `AttachUserPolicy`
- **Metric filter:** `attach-user-policy`
- **Metric:** `CloudSecurityMonitoring / AttachUserPolicyEvents`
- **Alarm:** `attach-user-policy-alarm`

---

## Why This Architecture Matters

This architecture is intentionally broader than a single host-based detection setup.

A host-only lab can show how one system is attacked and how its local logs can be analyzed. This architecture goes further by showing how a defender can monitor both:

- activity happening inside a Linux system
- activity happening across the AWS account control plane
- network-control-plane changes through security group events
- identity-control-plane changes through IAM events

That distinction matters because real cloud security monitoring depends on visibility across multiple telemetry sources, not just one log file on one machine.

---

## Architecture Summary

This lab combines host monitoring and cloud control-plane monitoring into a single AWS-native workflow.

The environment includes:

- one EC2 instance as the monitored system
- CloudWatch Agent for host log forwarding
- CloudWatch Logs for centralized log collection
- CloudTrail for AWS account activity logging
- S3 for retained CloudTrail storage
- CloudWatch metric filters for detection logic
- CloudWatch alarms for threshold-based alerting
- SNS for alert delivery

Together, these components support a basic but realistic cloud security monitoring pipeline.

---

## Architecture Diagram

```text
HOST-LEVEL MONITORING PATH

Test SSH Activity
        ↓
EC2 Linux Instance
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
        │     Trail Log Storage
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
