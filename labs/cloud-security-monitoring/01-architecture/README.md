# 01 - Architecture

## Purpose

This section defines the architecture for the cloud security monitoring lab.

The goal of this architecture is to centralize security-relevant telemetry from both the operating system and the AWS account, apply detection logic to that telemetry, and generate alerts when suspicious activity occurs.

This lab is designed to demonstrate a small but realistic AWS-native monitoring workflow rather than a single host-based detection mechanism.

---

## Architecture Goals

The architecture must support the following:

- collection of host-level logs from an EC2 instance
- collection of AWS control-plane logs through CloudTrail
- centralized visibility into multiple log sources
- detection of suspicious behavior using AWS-native services
- alerting when defined detection thresholds are met
- storage of log data for later review

---

## Core Components

### 1. EC2 Instance
A Linux EC2 instance will act as the monitored host.

This instance will be used to generate host-level telemetry, including authentication activity such as failed SSH login attempts. It represents the operating-system side of the environment.

### 2. CloudWatch Logs
CloudWatch Logs will be used to centralize host-level log data collected from the EC2 instance.

This will allow log data to be queried, filtered, and used for detection logic.

### 3. CloudTrail
CloudTrail will record AWS account activity and management events.

This provides visibility into control-plane actions such as console activity and API calls that would not appear in the operating-system logs of the EC2 instance.

### 4. Amazon S3
S3 will be used for log storage.

CloudTrail logs will be delivered to S3, providing retained records of AWS activity for later review and validation.

### 5. Detection Layer
Detection logic will be built using CloudWatch metric filters and alarms.

This layer will identify suspicious patterns in the ingested logs, such as repeated failed authentication attempts or specific CloudTrail events of interest.

### 6. Alerting Layer
Amazon SNS will be used to generate notifications when detection thresholds are met.

This represents the alerting stage of the monitoring workflow.

---

## Data Flow

The monitoring pipeline is expected to work as follows:

### Host-Level Monitoring Flow
1. Activity occurs on the EC2 instance
2. The EC2 instance generates operating-system log entries
3. Relevant logs are forwarded to CloudWatch Logs
4. CloudWatch metric filters evaluate the log stream
5. A CloudWatch alarm triggers if suspicious activity matches the detection logic
6. SNS sends an alert notification

### AWS Control-Plane Monitoring Flow
1. Activity occurs in the AWS account
2. CloudTrail records the management event
3. CloudTrail delivers logs to S3
4. Selected events are monitored for detection use cases
5. CloudWatch alarms trigger when detection criteria are met
6. SNS sends an alert notification

---

## Planned Detection Use Cases

The initial architecture is designed to support the following detections:

### Detection 1 - Repeated Failed SSH Logins
This detection will identify repeated failed authentication attempts against the EC2 instance.

### Detection 2 - Suspicious CloudTrail Activity
This detection will identify selected AWS management events that should be reviewed, such as high-risk account activity or administrative changes.

---

## Why This Architecture Matters

This architecture is intentionally broader than a single host-based detection setup.

A host-only lab can show how one system is attacked and how its local logs can be analyzed. This architecture goes further by showing how a defender can monitor both:

- activity happening inside a system
- activity happening across the AWS environment itself

That distinction is important because real cloud security monitoring depends on visibility across multiple telemetry sources, not just one log file on one machine.

---

## Architecture Summary

This lab combines host monitoring and cloud monitoring into a single AWS-native workflow.

The environment will include:

- one EC2 instance as the monitored system
- CloudWatch Logs for centralized host log collection
- CloudTrail for AWS account activity logging
- S3 for retained CloudTrail storage
- CloudWatch metric filters and alarms for detection
- SNS for alert delivery

Together, these components support a basic but realistic cloud security monitoring pipeline.

---

## Architecture Diagram

```mermaid
flowchart TD

    subgraph H1["Host-Level Monitoring Path"]
        A[Attacker / Test Activity<br/>Repeated SSH Attempts]
        B[EC2 Linux Instance<br/>/var/log/auth.log]
        C[CloudWatch Agent]
        D[CloudWatch Logs<br/>EC2 Auth Log Group]
        E[Metric Filter<br/>Failed SSH Login Pattern]
        F[CloudWatch Alarm]
        G[SNS Notification]
        A --> B --> C --> D --> E --> F --> G
    end

    subgraph H2["AWS Control-Plane Monitoring Path"]
        H[AWS Account Activity<br/>Console / API Events]
        I[CloudTrail]
        J[S3 Bucket<br/>Trail Log Storage]
        K[CloudWatch Logs<br/>CloudTrail Log Group]
        L[Metric Filter<br/>Suspicious CloudTrail Activity]
        M[CloudWatch Alarm]
        N[SNS Notification]
        H --> I
        I --> J
        I --> K
        K --> L --> M --> N
    end
