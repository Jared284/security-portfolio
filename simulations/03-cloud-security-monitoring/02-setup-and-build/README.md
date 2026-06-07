# 02 - Setup and Build

## Purpose

This section documents the AWS resources and configuration steps used to build the Cloud Security Monitoring Lab.

The goal of this phase was to create the telemetry foundation before adding attack simulation, detection logic, CloudWatch alarms, or alert validation.

This setup phase established the resources needed to collect:

- Host-level authentication logs from an EC2 instance
- AWS control-plane activity from CloudTrail
- Alert notifications through SNS

---

## Build Strategy

The environment was built in a deliberate order so each layer could support the next.

```text
Storage and alerting foundation
        ↓
CloudTrail logging
        ↓
EC2 monitored host
        ↓
CloudWatch Agent log forwarding
        ↓
Centralized log validation
        ↓
Detection engineering readiness
```

The build process followed this sequence:

1. Created an S3 bucket for CloudTrail log storage.
2. Created an SNS topic for alert delivery.
3. Created a CloudTrail trail for AWS account activity.
4. Configured CloudTrail to deliver logs to S3.
5. Configured CloudTrail to deliver logs to CloudWatch Logs.
6. Launched a Linux EC2 instance as the monitored host.
7. Attached an IAM role to the EC2 instance for CloudWatch Agent permissions.
8. Installed and configured the CloudWatch Agent on the EC2 instance.
9. Forwarded `/var/log/auth.log` from the EC2 instance to CloudWatch Logs.
10. Validated that both host-level logs and CloudTrail logs were being collected successfully.

---

## Core AWS Resources Built

### 1. S3 Bucket

An S3 bucket was created to store CloudTrail logs.

CloudTrail delivers AWS account activity logs to this bucket for retained storage and later review.

Security purpose:

- Preserve AWS control-plane audit logs
- Support later investigation
- Maintain retained evidence outside of CloudWatch Logs

---

### 2. SNS Topic

An SNS topic was created for alert delivery.

This topic was later connected to CloudWatch alarms so that security-relevant detections could generate email notifications.

SNS topic:

```text
cloud-security-monitoring-alerts
```

Security purpose:

- Provide an alert notification path
- Allow CloudWatch alarms to send email notifications
- Support end-to-end detection validation

---

### 3. CloudTrail Trail

A CloudTrail trail was created to capture AWS management activity.

CloudTrail was configured to deliver logs to:

- **S3** for retained storage
- **CloudWatch Logs** for monitoring and detection

CloudTrail trail:

```text
cloud-security-monitoring-trail
```

Security purpose:

- Record AWS control-plane activity
- Capture security group changes
- Capture IAM policy attachment events
- Provide audit evidence for cloud-side detections

---

### 4. CloudWatch Log Groups

CloudWatch Logs was used to centralize both host-level logs and AWS control-plane logs.

Primary log groups:

```text
cloud-security-monitoring-auth
cloud-security-monitoring-cloudtrail
```

The `cloud-security-monitoring-auth` log group stores Linux authentication logs from the EC2 instance.

The `cloud-security-monitoring-cloudtrail` log group stores AWS control-plane events delivered by CloudTrail.

Security purpose:

- Centralize security telemetry
- Support CloudWatch Logs metric filters
- Provide log visibility for validation and troubleshooting

---

### 5. EC2 Linux Instance

An Ubuntu EC2 instance was launched as the monitored host.

The instance generated host-level telemetry for the lab, including SSH authentication events written to:

```text
/var/log/auth.log
```

Security purpose:

- Provide a workload that generates host-level security logs
- Support SSH authentication monitoring
- Create a controlled source of Linux telemetry

---

### 6. IAM Role for EC2

An IAM role was attached to the EC2 instance so the CloudWatch Agent could publish logs to CloudWatch Logs.

EC2 role:

```text
cloud-security-monitoring-ec2-role
```

Managed policy attached:

```text
CloudWatchAgentServerPolicy
```

Security purpose:

- Allow the EC2 instance to send logs to CloudWatch Logs
- Avoid hardcoding AWS credentials on the instance
- Use role-based access instead of static access keys

---

### 7. CloudWatch Agent

The CloudWatch Agent was installed and configured on the EC2 instance.

The agent forwards the following host log file:

```text
/var/log/auth.log
```

to the CloudWatch Logs log group:

```text
cloud-security-monitoring-auth
```

Security purpose:

- Move host-level authentication telemetry into centralized logging
- Enable metric filters against Linux authentication events
- Support detection of repeated invalid-user SSH attempts

---

## Completed Build Objectives

By the end of this phase, the lab had:

- A working Ubuntu EC2 monitored host
- An IAM role attached to the EC2 instance for CloudWatch Agent permissions
- CloudTrail enabled for AWS account activity
- CloudTrail logs delivered to S3
- CloudTrail logs delivered to CloudWatch Logs
- Linux authentication logs forwarded from EC2 to CloudWatch Logs
- An SNS topic prepared for alert delivery
- A stable telemetry pipeline ready for detection engineering

---

## Validation Completed

Before moving into attack simulation and detection engineering, I confirmed that:

- CloudTrail was actively recording AWS account activity
- CloudTrail logs were delivered to S3
- CloudTrail logs were visible in CloudWatch Logs
- EC2 host authentication logs were visible in CloudWatch Logs
- The CloudWatch Agent was forwarding `/var/log/auth.log`
- The logging pipeline was stable enough to support metric filters, alarms, and SNS notifications

---

## Evidence Collected

The setup and build phase was supported by screenshots showing resource creation and initial telemetry validation.

Evidence includes:

- EC2 instance running successfully
- IAM role attached to the EC2 instance
- CloudWatch Agent installed and configured
- CloudWatch log groups created
- Host authentication logs visible in CloudWatch Logs
- CloudTrail trail configured
- CloudTrail logs delivered to CloudWatch Logs
- SNS topic created for alert delivery

These screenshots confirm that the core AWS resources were built and that telemetry was flowing before detection logic was added.

---

## Final Setup Outcome

This phase created the foundation for the rest of the lab.

At the end of setup, the environment could collect and centralize:

- Linux authentication activity from the EC2 instance
- AWS control-plane activity from CloudTrail

This allowed the next phases of the lab to focus on attack simulation, detection engineering, alarm configuration, and alert validation.
