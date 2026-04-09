# Cloud Security Monitoring

## Overview

This lab demonstrates centralized cloud security monitoring in AWS using two different telemetry sources:

- **Host-level Linux authentication logs** from an EC2 instance
- **AWS control-plane activity** captured through CloudTrail

The goal of the project is to show that security-relevant events from both a Linux host and the AWS environment itself can be centralized, monitored, and used as the basis for detection and alerting.

This lab is designed to be more than a basic logging demo. The focus is on building a monitoring pipeline that is understandable, testable, and defensible in an interview setting.

## Why I Built This Lab

I built this project to strengthen the cloud-security side of my portfolio and show that I can work across multiple layers of security monitoring in AWS.

This lab is meant to demonstrate that I can:

- collect and centralize telemetry from multiple sources
- monitor both host activity and cloud control-plane activity
- engineer basic detections from raw logs
- configure threshold-based alerting
- validate security controls end to end
- troubleshoot failures across the monitoring pipeline

Within my broader portfolio, this lab complements other projects by adding cloud-native monitoring depth.

## What This Lab Is Supposed to Prove

This project is intended to prove that I can build and explain a centralized monitoring workflow in AWS.

Specifically, it shows that I can:

- forward Linux authentication logs from an EC2 instance into CloudWatch
- centralize CloudTrail telemetry for AWS account activity
- create detection logic from real log patterns
- turn those detections into CloudWatch metrics and alarms
- route alerts through Amazon SNS
- validate the entire path from attack activity to notification delivery

## Telemetry Sources

This lab currently focuses on two telemetry sources.

### 1. Host-Level Authentication Logs

The EC2 monitoring host generates Linux SSH authentication telemetry in:

```text
/var/log/auth.log
```

These logs are forwarded to CloudWatch Logs using the CloudWatch Agent.

This telemetry is used to detect repeated invalid-user SSH login attempts.

### 2. AWS Control-Plane Activity

AWS API and management activity is collected through CloudTrail.

CloudTrail is configured to send logs to:

- **S3** for storage
- **CloudWatch Logs** for monitoring

This creates the foundation for cloud-side detections involving AWS administrative actions and account activity.

## Lab Architecture

At a high level, the lab works like this:

1. An EC2 Ubuntu instance generates host-level authentication logs
2. The CloudWatch Agent forwards `/var/log/auth.log` to CloudWatch Logs
3. CloudTrail collects AWS control-plane activity and also forwards it to CloudWatch Logs
4. CloudWatch metric filters turn selected log patterns into custom metrics
5. CloudWatch alarms evaluate those metrics against defined thresholds
6. Amazon SNS sends notifications when alarm conditions are met

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

## Current Status

### Completed Build Components

The following components have already been built:

- S3 bucket for CloudTrail log storage
- SNS topic for alert delivery
- CloudTrail trail for AWS activity logging
- CloudTrail delivery into S3 and CloudWatch Logs
- EC2 Ubuntu monitoring host
- IAM role for EC2 CloudWatch access
- CloudWatch Agent installation and configuration
- Forwarding of `/var/log/auth.log` into CloudWatch Logs
- Custom metric filter for invalid-user SSH attempts
- CloudWatch alarm for repeated invalid-user SSH activity
- SNS-based email notification path

### Completed Validation Work

The host-based monitoring path has been validated end to end.

This validation included:

- generating repeated fake SSH login attempts against the EC2 host
- confirming `Invalid user` entries in `/var/log/auth.log`
- confirming those log entries in CloudWatch Logs
- confirming the custom metric `InvalidUserSSHAttempts`
- confirming the CloudWatch alarm entered the `ALARM` state
- confirming SNS email alert delivery

## Detection Implemented So Far

### Host-Based Detection

The first implemented detection focuses on repeated invalid-user SSH login attempts against the EC2 instance.

This detection uses:

- **Log source:** `/var/log/auth.log`
- **Filter pattern:** `"Invalid user"`
- **Metric namespace:** `CloudSecurityMonitoring`
- **Metric name:** `InvalidUserSSHAttempts`

This metric is then used to trigger a CloudWatch alarm when the threshold is exceeded.

## Screenshots and Evidence

Each major section of the lab includes evidence showing the build and validation process.

Examples include:

- attacker-side SSH attempts
- host log validation
- CloudWatch log ingestion
- metric filter configuration
- metric validation
- CloudWatch alarm state changes
- SNS email notification delivery

The goal of the screenshots is to support technical claims with direct evidence rather than screenshots for their own sake.

## What Has Been Finished vs. What Comes Next

### Finished So Far

The host-side monitoring and alerting path is complete and validated:

- attack simulation
- host telemetry generation
- centralized log ingestion
- detection engineering
- alarm triggering
- SNS notification delivery

### Still Planned

The lab is not finished yet.

The major remaining area is the **CloudTrail-side detection path**, which will extend monitoring beyond the Linux host and deeper into AWS account activity.

Planned next steps include building detections for cloud-side behaviors such as:

- security group modifications
- failed console logins
- IAM changes
- other suspicious control-plane actions

The final phase of the lab will also include a reflections and improvements section covering limitations, future enhancements, and design tradeoffs.

## Key Takeaway

This lab is intended to show that I can build and validate a cloud-native monitoring pipeline in AWS, not just configure services blindly.

So far, it demonstrates that I can move from raw host telemetry to centralized logs, custom detections, alarms, and real notification delivery. The next step is to extend that same approach to CloudTrail-based AWS activity monitoring so the lab covers both host and cloud control-plane visibility.
