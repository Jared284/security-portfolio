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

This lab focuses on two telemetry sources.

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

This telemetry is used to detect security-relevant AWS administrative activity, including security group ingress rule addition and removal events.

## Lab Architecture

At a high level, the lab works like this:

1. An EC2 Ubuntu instance generates host-level authentication logs
2. The CloudWatch Agent forwards `/var/log/auth.log` to CloudWatch Logs
3. CloudTrail collects AWS control-plane activity and also forwards it to CloudWatch Logs
4. CloudWatch metric filters turn selected log patterns into custom metrics
5. CloudWatch alarms evaluate those metrics against defined thresholds
6. Amazon SNS sends notifications when alarm conditions are met

This creates two parallel monitoring paths inside the same lab:

- a **host-side path** based on Linux authentication telemetry
- a **cloud-side path** based on CloudTrail control-plane events

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

The following components have been built:

- S3 bucket for CloudTrail log storage
- SNS topic for alert delivery
- CloudTrail trail for AWS activity logging
- CloudTrail delivery into S3 and CloudWatch Logs
- EC2 Ubuntu monitoring host
- IAM role for EC2 CloudWatch access
- CloudWatch Agent installation and configuration
- forwarding of `/var/log/auth.log` into CloudWatch Logs
- custom metric filter for invalid-user SSH attempts
- CloudWatch alarm for repeated invalid-user SSH activity
- custom metric filter for `RevokeSecurityGroupIngress` CloudTrail events
- CloudWatch alarm for security group ingress rule removal activity
- custom metric filter for `AuthorizeSecurityGroupIngress` CloudTrail events
- CloudWatch alarm for security group ingress rule addition activity
- SNS-based email notification path

### Completed Validation Work

Both the host-side and cloud-side monitoring paths have now been validated end to end.

### Host-Side Validation

The host-based monitoring path was validated by:

- generating repeated fake SSH login attempts against the EC2 host
- confirming `Invalid user` entries in `/var/log/auth.log`
- confirming those log entries in CloudWatch Logs
- confirming the custom metric `InvalidUserSSHAttempts`
- confirming the CloudWatch alarm entered the `ALARM` state
- confirming SNS email alert delivery

### CloudTrail-Side Validation

The CloudTrail-based monitoring path was validated by:

- making controlled security group rule changes on the EC2 instance’s attached security group
- identifying the resulting CloudTrail events `AuthorizeSecurityGroupIngress` and `RevokeSecurityGroupIngress`
- confirming those events in CloudTrail Event history
- confirming the events were matched by CloudWatch metric filters
- confirming the custom metrics `AuthorizeSecurityGroupIngressEvents` and `RevokeSecurityGroupIngressEvents`
- confirming the CloudWatch alarms entered the `ALARM` state
- confirming SNS email alert delivery

## Detection Implemented So Far

### 1. Host-Based Detection

The first implemented detection focuses on repeated invalid-user SSH login attempts against the EC2 instance.

This detection uses:

- **Log source:** `/var/log/auth.log`
- **Filter pattern:** `"Invalid user"`
- **Metric namespace:** `CloudSecurityMonitoring`
- **Metric name:** `InvalidUserSSHAttempts`

This metric is used to trigger a CloudWatch alarm when repeated invalid-user SSH activity exceeds the configured threshold.

### 2. CloudTrail-Based Detection: RevokeSecurityGroupIngress

The second implemented detection focuses on AWS security group ingress rule removal activity.

This detection uses:

- **Telemetry source:** CloudTrail
- **CloudWatch Logs log group:** `cloud-security-monitoring-cloudtrail`
- **Event source:** `ec2.amazonaws.com`
- **Event name:** `RevokeSecurityGroupIngress`
- **Metric namespace:** `CloudSecurityMonitoring`
- **Metric name:** `RevokeSecurityGroupIngressEvents`

This metric is used to trigger a CloudWatch alarm when a matching security group ingress rule removal event is detected.

### 3. CloudTrail-Based Detection: AuthorizeSecurityGroupIngress

The third implemented detection focuses on AWS security group ingress rule addition activity.

This detection uses:

- **Telemetry source:** CloudTrail
- **CloudWatch Logs log group:** `cloud-security-monitoring-cloudtrail`
- **Event source:** `ec2.amazonaws.com`
- **Event name:** `AuthorizeSecurityGroupIngress`
- **Metric namespace:** `CloudSecurityMonitoring`
- **Metric name:** `AuthorizeSecurityGroupIngressEvents`

This metric is used to trigger a CloudWatch alarm when a matching security group ingress rule addition event is detected.

## Alerting Implemented So Far

The lab currently includes three CloudWatch alarms:

### Host-Side Alarm

- **Alarm name:** `invalid-user-ssh-attempts-alarm`
- **Signal:** repeated invalid-user SSH attempts
- **Threshold:** greater than `3` within `5` minutes

### CloudTrail-Side Revoke Alarm

- **Alarm name:** `revoke-security-group-ingress-alarm`
- **Signal:** `RevokeSecurityGroupIngress` CloudTrail events
- **Threshold:** greater than or equal to `1` within `5` minutes

### CloudTrail-Side Authorize Alarm

- **Alarm name:** `authorize-security-group-ingress-alarm`
- **Signal:** `AuthorizeSecurityGroupIngress` CloudTrail events
- **Threshold:** greater than or equal to `1` within `5` minutes

All three alarms publish notifications to the SNS topic:

```text
cloud-security-monitoring-alerts
```

## Screenshots and Evidence

Each major section of the lab includes evidence showing the build and validation process.

Examples include:

- attacker-side SSH attempts
- host log validation
- CloudWatch log ingestion
- CloudTrail event validation
- metric filter configuration
- custom metric validation
- CloudWatch alarm state changes
- SNS email notification delivery

The goal of the screenshots is to support technical claims with direct evidence rather than screenshots for their own sake.

## What Has Been Finished vs. What Comes Next

### Finished So Far

The lab now has three validated monitoring and alerting paths:

#### Host-side path
- attack simulation
- host telemetry generation
- centralized log ingestion
- detection engineering
- alarm triggering
- SNS notification delivery

#### CloudTrail-side path: ingress rule removal
- CloudTrail event generation
- CloudTrail event validation
- CloudWatch log ingestion
- detection engineering
- alarm triggering
- SNS notification delivery

#### CloudTrail-side path: ingress rule addition
- CloudTrail event generation
- CloudTrail event validation
- CloudWatch log ingestion
- detection engineering
- alarm triggering
- SNS notification delivery

### Still Planned

The lab is much stronger now, but it is still not final.

The next logical improvements include:

- adding more CloudTrail-side detections beyond security group ingress additions and removals
- expanding cloud-side coverage to other high-signal AWS administrative events
- adding more host-side detections beyond invalid-user SSH activity
- improving cross-source correlation between host and cloud telemetry
- expanding the reflections and improvements section as the lab matures

Strong future candidates include detections for:

- failed AWS console login activity
- IAM policy, role, or user changes
- suspicious administrative API activity
- trail modification or disablement attempts

## Key Takeaway

This lab now demonstrates multiple cloud security monitoring paths in AWS.

On the host side, it shows how Linux authentication telemetry can be collected, centralized, converted into a custom detection metric, and escalated into an alert.

On the cloud side, it shows how AWS control-plane activity captured by CloudTrail can be turned into custom security metrics for both security group ingress rule additions and removals, evaluated by CloudWatch alarms, and delivered through SNS.

Together, these paths show that I can build and validate a cloud-native monitoring workflow in AWS across both workload-level and control-plane visibility.
