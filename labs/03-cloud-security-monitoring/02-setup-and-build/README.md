# 02 - Setup and Build

## Purpose

This section documents the AWS resources and configuration steps used to build the cloud security monitoring lab.

The goal of this phase is to create the monitoring environment first, before any attack simulation or detection logic is added.

---

## Build Strategy

The environment will be built in the following order:

1. Create an S3 bucket for CloudTrail log storage
2. Create an SNS topic for alert delivery
3. Create a CloudTrail trail
4. Configure CloudTrail to send logs to S3
5. Configure CloudTrail to send logs to CloudWatch Logs
6. Launch a Linux EC2 instance
7. Install and configure the CloudWatch Agent on the EC2 instance
8. Forward host-level logs to CloudWatch Logs
9. Validate that both host logs and CloudTrail logs are being collected successfully

---

## Core Resources to Build

### 1. S3 Bucket
This bucket will store CloudTrail logs for retention and later review.

### 2. SNS Topic
This topic will be used later for alert notifications when detections trigger.

### 3. CloudTrail Trail
This trail will capture AWS management activity and deliver logs to both S3 and CloudWatch Logs.

### 4. CloudWatch Log Groups
CloudWatch Logs will centralize:
- EC2 authentication / host logs
- CloudTrail management event logs

### 5. EC2 Linux Instance
This instance will act as the monitored host and will generate host-level telemetry during the lab.

### 6. CloudWatch Agent
The CloudWatch Agent will forward host-level logs from the EC2 instance into CloudWatch Logs.

---

## Build Objectives

By the end of this phase, the lab should have:

- a working EC2 instance
- CloudTrail enabled
- CloudTrail logs stored in S3
- CloudTrail logs available in CloudWatch Logs
- host-level logs forwarded from EC2 to CloudWatch Logs
- a prepared SNS topic for future alerting

---

## Validation Goals

Before moving to attack simulation or detection engineering, the following must be confirmed:

- CloudTrail is actively recording AWS account activity
- CloudTrail logs are arriving in S3
- CloudTrail logs are visible in CloudWatch Logs
- EC2 host logs are visible in CloudWatch Logs
- the logging pipeline is stable and working as expected

---

## Notes

Detection logic and alarms will not be created during this phase.

This phase is only focused on building and validating the telemetry pipeline.
