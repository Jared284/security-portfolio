# 02 - Setup and Build

## Purpose

This section documents the AWS resources and configuration steps used to build the cloud security monitoring lab.

The goal of this phase was to create the monitoring environment before adding attack simulation, detection logic, alarms, or alert validation.

---

## Build Strategy

The environment was built in the following order:

1. Create an S3 bucket for CloudTrail log storage
2. Create an SNS topic for alert delivery
3. Create a CloudTrail trail
4. Configure CloudTrail to send logs to S3
5. Configure CloudTrail to send logs to CloudWatch Logs
6. Launch a Linux EC2 instance
7. Attach an IAM role to the EC2 instance for CloudWatch Agent access
8. Install and configure the CloudWatch Agent on the EC2 instance
9. Forward host-level authentication logs to CloudWatch Logs
10. Validate that both host logs and CloudTrail logs were being collected successfully

---

## Core Resources Built

### 1. S3 Bucket

An S3 bucket was used to store CloudTrail logs for retention and later review.

CloudTrail delivers AWS account activity logs to this bucket, providing retained evidence of control-plane activity.

### 2. SNS Topic

An SNS topic was created for alert notifications.

This topic was later connected to CloudWatch alarms so that security-relevant events could generate email notifications.

The SNS topic used in this lab was:

```text
cloud-security-monitoring-alerts
```

### 3. CloudTrail Trail

A CloudTrail trail was created to capture AWS management activity.

The trail was configured to deliver logs to:

- S3 for retained storage
- CloudWatch Logs for monitoring and detection

The CloudTrail trail used in this lab was:

```text
cloud-security-monitoring-trail
```

### 4. CloudWatch Log Groups

CloudWatch Logs was used to centralize both host-level logs and AWS control-plane logs.

The main log groups used in this lab were:

```text
cloud-security-monitoring-auth
cloud-security-monitoring-cloudtrail
```

The `cloud-security-monitoring-auth` log group stores Linux authentication logs from the EC2 instance.

The `cloud-security-monitoring-cloudtrail` log group stores AWS control-plane events delivered by CloudTrail.

### 5. EC2 Linux Instance

A Linux EC2 instance was launched as the monitored host.

This instance generated host-level telemetry for the lab, including SSH authentication events written to:

```text
/var/log/auth.log
```

The EC2 instance was also used as the environment for validating CloudWatch Agent log forwarding.

### 6. IAM Role for EC2

An IAM role was attached to the EC2 instance so the CloudWatch Agent could send logs to CloudWatch.

The EC2 role used in this lab was:

```text
cloud-security-monitoring-ec2-role
```

The role had the AWS managed policy:

```text
CloudWatchAgentServerPolicy
```

This allowed the EC2 instance to publish log data to CloudWatch Logs.

### 7. CloudWatch Agent

The CloudWatch Agent was installed and configured on the EC2 instance.

The agent was configured to forward the following host log file:

```text
/var/log/auth.log
```

to the CloudWatch Logs log group:

```text
cloud-security-monitoring-auth
```

---

## Build Objectives Completed

By the end of this phase, the lab had:

- a working EC2 Linux instance
- an IAM role attached to the EC2 instance for CloudWatch Agent permissions
- CloudTrail enabled
- CloudTrail logs stored in S3
- CloudTrail logs delivered to CloudWatch Logs
- host-level authentication logs forwarded from EC2 to CloudWatch Logs
- an SNS topic prepared for alert delivery
- a stable telemetry pipeline ready for detection engineering

---

## Validation Completed

Before moving into attack simulation and detection engineering, the following were confirmed:

- CloudTrail was actively recording AWS account activity
- CloudTrail logs were delivered to S3
- CloudTrail logs were visible in CloudWatch Logs
- EC2 host authentication logs were visible in CloudWatch Logs
- the CloudWatch Agent was successfully forwarding `/var/log/auth.log`
- the logging pipeline was stable enough to support metric filters, alarms, and SNS notifications

---

## Evidence

The setup and build phase was supported by screenshots showing resource creation and telemetry validation, including:

- EC2 instance status
- IAM role attachment
- CloudWatch Agent setup
- CloudWatch log group creation
- host authentication logs in CloudWatch
- CloudTrail event delivery
- SNS topic configuration

Example evidence files include:

```text
ec2-instance-running.png
ec2-instance-iam-role-attached.png
cloudwatch-agent-running.png
cloudwatch-auth-log-group-validation.png
cloudtrail-log-group-events.png
sns-topic-confirmed-subscription-icloud.png
```

---

## Notes

Detection logic and alarms were created after this setup phase.

This phase focused on building and validating the telemetry foundation. The later detection and alerting sections build on this setup by creating CloudWatch metric filters, CloudWatch alarms, and SNS alert delivery paths.
