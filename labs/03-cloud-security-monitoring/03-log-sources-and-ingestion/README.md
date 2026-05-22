# 03 - Log Sources and Ingestion

## Purpose

This section documents the log sources used in the AWS Cloud Security Monitoring Lab and how those logs were centralized into CloudWatch Logs.

The goal of this phase was to prove that security-relevant telemetry was being collected reliably before building detections, alarms, or alerting workflows.

The lab collects telemetry from two layers:

- **Host-level activity** from the EC2 monitoring host
- **AWS control-plane activity** from CloudTrail

This ingestion layer is the foundation for the rest of the lab.

---

## Telemetry Sources

The lab uses two primary telemetry sources.

---

## Source 1: Host-Level Authentication Logs

The EC2 Ubuntu monitoring host generates Linux authentication telemetry in:

```text
/var/log/auth.log
```

These logs include SSH-related events such as:

- Invalid-user login attempts
- Failed authentication attempts
- Accepted logins
- SSH daemon authentication messages

For this lab, the most important host-side signal was:

```text
Invalid user
```

This signal was later used to detect repeated invalid-user SSH login attempts against the EC2 instance.

---

## Source 2: AWS Control-Plane Activity

AWS administrative and API activity is collected through CloudTrail.

CloudTrail records management-plane activity such as:

- API calls made against AWS services
- Security group configuration changes
- IAM permission changes
- Resource enumeration actions
- Other AWS account management operations

For this lab, the most important CloudTrail events were:

```text
RevokeSecurityGroupIngress
AuthorizeSecurityGroupIngress
AttachUserPolicy
```

These events provided visibility into:

- Network-control-plane changes through security group events
- Identity-control-plane changes through IAM events

---

## Ingestion Architecture

The lab uses two separate ingestion paths depending on the telemetry source.

---

## Path 1: Host-Side Log Ingestion

The EC2 instance uses the CloudWatch Agent to forward Linux authentication logs into CloudWatch Logs.

### Flow

1. SSH activity occurs against the EC2 instance.
2. The EC2 instance writes authentication events to `/var/log/auth.log`.
3. The CloudWatch Agent reads the log file.
4. The agent forwards the log data into CloudWatch Logs.
5. The logs become available for centralized monitoring and detection engineering.

### Destination Log Group

```text
cloud-security-monitoring-auth
```

### Security Purpose

This ingestion path allows Linux authentication activity to be monitored centrally instead of only being available on the host itself.

---

## Path 2: CloudTrail Log Ingestion

CloudTrail records AWS control-plane activity and delivers it into both S3 and CloudWatch Logs.

### Flow

1. An AWS administrative action occurs.
2. CloudTrail records the management event.
3. CloudTrail delivers the event to S3 for retained storage.
4. CloudTrail also delivers the event to CloudWatch Logs for monitoring and detection.
5. The event becomes available for CloudWatch Logs metric filters.

### Destination Log Group

```text
cloud-security-monitoring-cloudtrail
```

### Security Purpose

This ingestion path allows AWS account activity to be monitored and used for cloud-side detections.

---

## Log Groups Used

| Log Group | Telemetry Source | Purpose |
|---|---|---|
| `cloud-security-monitoring-auth` | EC2 `/var/log/auth.log` | Host-level SSH authentication monitoring |
| `cloud-security-monitoring-cloudtrail` | AWS CloudTrail | AWS control-plane monitoring |

---

## Supporting AWS Components

| Component | Name / Configuration | Purpose |
|---|---|---|
| EC2 instance | `cloud-security-monitoring-host` | Generates host-level authentication telemetry |
| Host log file | `/var/log/auth.log` | Stores Linux authentication events |
| IAM role | `cloud-security-monitoring-ec2-role` | Allows EC2 to publish logs through CloudWatch Agent |
| Managed policy | `CloudWatchAgentServerPolicy` | Grants CloudWatch Agent log publishing permissions |
| CloudTrail trail | `cloud-security-monitoring-trail` | Records AWS control-plane activity |
| S3 bucket | CloudTrail log storage bucket | Stores retained CloudTrail logs |
| CloudWatch Logs | Two log groups | Centralizes host and AWS telemetry |

---

## Why These Sources Were Chosen

These two sources were chosen because they provide visibility across two important security layers.

### Host-Level Visibility

Linux authentication logs show activity occurring inside the monitored workload.

This includes:

- SSH login attempts
- Invalid usernames
- Failed authentication activity
- Successful authentication activity

### AWS Control-Plane Visibility

CloudTrail logs show activity occurring across the AWS account.

This includes:

- Security group changes
- IAM policy changes
- AWS API activity
- Administrative actions

Together, these sources make the lab stronger than a single-source logging project because they demonstrate monitoring across:

- Workload-level behavior
- Cloud network-control-plane behavior
- Cloud identity-control-plane behavior

---

## Ingestion Validation

I validated ingestion separately for both telemetry sources.

---

## Host-Side Ingestion Validation

I confirmed that Linux authentication activity from `/var/log/auth.log` appeared in the CloudWatch log group:

```text
cloud-security-monitoring-auth
```

This proved that:

- The EC2 instance was generating authentication telemetry
- The CloudWatch Agent was installed and running
- Host logs were reaching CloudWatch Logs successfully
- The host-side telemetry path was ready for metric filters

---

## CloudTrail-Side Ingestion Validation

I confirmed that AWS administrative activity appeared in the CloudWatch log group:

```text
cloud-security-monitoring-cloudtrail
```

I also confirmed specific CloudTrail events through Event history:

```text
RevokeSecurityGroupIngress
AuthorizeSecurityGroupIngress
AttachUserPolicy
```

This proved that:

- CloudTrail was recording AWS control-plane activity correctly
- CloudTrail delivery into CloudWatch Logs was working
- Security group events were available for detection engineering
- IAM policy attachment events were available for detection engineering
- The cloud-side telemetry path was ready for metric filters

---

## Evidence

### Host Authentication Logs in CloudWatch

![CloudWatch log entries showing forwarded host authentication events](../screenshots/cloudwatch-auth-log-invalid-user-events.png)

### CloudTrail Events in CloudWatch Logs

![CloudTrail log group showing control-plane event records in CloudWatch](../screenshots/cloudtrail-log-group-events.png)

### CloudTrail IAM Event Validation

![CloudTrail Event history showing AttachUserPolicy event](../screenshots/cloudtrail-attach-user-policy-event.png)

---

## How This Supports Later Phases

This phase created the ingestion layer required for the rest of the lab.

Later phases depend on this telemetry foundation:

| Later Phase | Dependency on This Section |
|---|---|
| `04-attack-simulation` | Requires host-side telemetry to be generated and collected |
| `05-detection-engineering` | Requires populated CloudWatch log groups for metric filters |
| `06-alerting-and-response` | Requires detections to produce custom metrics and alarms |

Without this ingestion phase, the rest of the monitoring workflow would not have reliable data to detect or alert on.

---

## Limitations

The ingestion layer is functional but intentionally small.

Current limitations include:

- Host-side ingestion focuses on one Linux log source
- Cloud-side ingestion focuses on selected CloudTrail management events
- Logs are centralized, but not normalized into a broader SIEM platform
- Correlation between host-level and CloudTrail telemetry is not implemented yet

These limitations are acceptable for the scope of this lab because the goal was to build and validate a focused AWS-native monitoring workflow.

---

## Key Takeaway

This phase established the telemetry foundation for the lab by centralizing both Linux host authentication logs and AWS control-plane activity into CloudWatch Logs.

That made it possible to build detections and alerts on top of:

- Host-level authentication events
- Cloud network-control-plane events
- Cloud identity-control-plane events

This is what allowed the lab to evolve from basic log collection into a real cloud security monitoring workflow.
