# 06 - Alerting and Response

## Purpose

This section documents how the custom detection metrics created earlier in the lab were converted into CloudWatch alarms and SNS notifications.

The goal of this phase was to prove that the lab could move beyond detection and complete the full path from security event to alert delivery.

This phase validates four alerting paths:

1. Host-side alerting for repeated invalid-user SSH attempts
2. CloudTrail-side alerting for security group ingress rule removal
3. CloudTrail-side alerting for security group ingress rule addition
4. CloudTrail-side alerting for IAM managed policy attachment

Together, these alerting paths show that host activity, AWS network-control-plane activity, and AWS identity-control-plane activity can be converted into real notifications.

---

## Alerting Scope

| Alert | Metric | Source | Threshold | Notification |
|---|---|---|---|---|
| Invalid SSH user attempts | `InvalidUserSSHAttempts` | EC2 `/var/log/auth.log` | `> 3` within 5 minutes | SNS email |
| Security group ingress rule removal | `RevokeSecurityGroupIngressEvents` | CloudTrail | `>= 1` within 5 minutes | SNS email |
| Security group ingress rule addition | `AuthorizeSecurityGroupIngressEvents` | CloudTrail | `>= 1` within 5 minutes | SNS email |
| IAM managed policy attachment | `AttachUserPolicyEvents` | CloudTrail | `>= 1` within 5 minutes | SNS email |

All alarms published notifications to the SNS topic:

```text
cloud-security-monitoring-alerts
```

---

## Alerting Workflow

Each alert followed the same workflow:

```text
Security-relevant activity
        ↓
Log or CloudTrail event generated
        ↓
CloudWatch Logs metric filter matches event
        ↓
Custom CloudWatch metric receives datapoint
        ↓
CloudWatch alarm evaluates threshold
        ↓
Alarm enters ALARM state
        ↓
SNS sends email notification
```

This workflow validated the full detection-to-notification pipeline.

---

## SNS Notification Path

Amazon SNS was used as the alert delivery mechanism.

SNS topic:

```text
cloud-security-monitoring-alerts
```

A confirmed email subscription was attached to this topic so CloudWatch alarm state changes could be delivered to an email inbox.

This SNS topic was used by all four CloudWatch alarms.

---

## Alert 1: Invalid SSH User Attempts

### Alerting Goal

This alert detects repeated invalid-user SSH login attempts against the EC2 monitoring host.

It is based on host-level Linux authentication telemetry.

---

### Metric Used

```text
CloudSecurityMonitoring / InvalidUserSSHAttempts
```

This metric counts each `Invalid user` SSH authentication event observed in forwarded Linux authentication logs.

---

### Alarm Configuration

```text
Alarm name: invalid-user-ssh-attempts-alarm
Metric: InvalidUserSSHAttempts
Namespace: CloudSecurityMonitoring
Threshold: Greater than 3
Evaluation window: 1 datapoint within 5 minutes
Alarm action: Publish to SNS topic cloud-security-monitoring-alerts
```

This means that if more than three invalid-user SSH attempts occur during the evaluation period, the alarm enters the `ALARM` state.

---

### Validation

I validated this alert by generating repeated invalid-user SSH attempts against the EC2 instance.

Validation confirmed that:

- The `InvalidUserSSHAttempts` metric reached a value of `5`
- The metric exceeded the configured threshold of `3`
- The alarm entered the `ALARM` state
- CloudWatch published the alarm notification to SNS
- SNS delivered the email notification successfully

---

### Operational Meaning

This alert indicates that the EC2 monitoring host received repeated SSH login attempts involving invalid usernames within a short time window.

From an analyst perspective, this could represent:

- Internet background scanning
- Reconnaissance against an exposed SSH service
- Early-stage brute-force activity
- Username-enumeration attempts

Follow-up investigation could include:

- Reviewing source IP addresses
- Checking whether the activity is isolated or recurring
- Comparing with other authentication telemetry
- Reviewing security group exposure
- Determining whether additional hardening or blocking is needed

---

## Alert 2: Security Group Ingress Rule Removal

### Alerting Goal

This alert detects when an ingress rule is removed from an AWS security group.

It is based on AWS control-plane activity captured by CloudTrail.

---

### Metric Used

```text
CloudSecurityMonitoring / RevokeSecurityGroupIngressEvents
```

This metric counts CloudTrail events where an ingress rule is removed from a security group.

---

### Alarm Configuration

```text
Alarm name: revoke-security-group-ingress-alarm
Metric: RevokeSecurityGroupIngressEvents
Namespace: CloudSecurityMonitoring
Threshold: Greater than or equal to 1
Evaluation window: 1 datapoint within 5 minutes
Alarm action: Publish to SNS topic cloud-security-monitoring-alerts
```

A single `RevokeSecurityGroupIngress` event within the evaluation period is enough to trigger the alarm.

---

### Why the Threshold Is Different

The SSH alarm uses a burst threshold because one failed login attempt is usually not meaningful on its own.

For a security group rule change, a single event is already security-relevant because it modifies AWS network access rules.

That makes `>= 1` appropriate for this alert.

---

### Validation

I validated this alert by removing a temporary ingress rule from the EC2 instance’s attached security group.

Validation confirmed that:

- CloudTrail recorded the `RevokeSecurityGroupIngress` event
- The CloudWatch Logs metric filter matched the event
- The custom metric `RevokeSecurityGroupIngressEvents` received a datapoint
- The alarm entered the `ALARM` state
- CloudWatch published the alarm notification to SNS
- SNS delivered the email notification successfully

---

### Operational Meaning

This alert indicates that an ingress rule was removed from an AWS security group.

From an analyst perspective, this could represent:

- Legitimate administrative maintenance
- Security hardening
- Unauthorized modification of network controls
- Unplanned changes to the exposed attack surface of a resource

Follow-up investigation could include:

- Confirming who made the change
- Reviewing the affected security group
- Identifying associated resources
- Determining whether the change was expected or approved
- Checking for related AWS activity before and after the event

---

## Alert 3: Security Group Ingress Rule Addition

### Alerting Goal

This alert detects when an ingress rule is added to an AWS security group.

It is based on AWS control-plane activity captured by CloudTrail.

---

### Metric Used

```text
CloudSecurityMonitoring / AuthorizeSecurityGroupIngressEvents
```

This metric counts CloudTrail events where an ingress rule is added to a security group.

---

### Alarm Configuration

```text
Alarm name: authorize-security-group-ingress-alarm
Metric: AuthorizeSecurityGroupIngressEvents
Namespace: CloudSecurityMonitoring
Threshold: Greater than or equal to 1
Evaluation window: 1 datapoint within 5 minutes
Alarm action: Publish to SNS topic cloud-security-monitoring-alerts
```

A single `AuthorizeSecurityGroupIngress` event within the evaluation period is enough to trigger the alarm.

---

### Why This Alert Matters

Security group ingress additions can increase the attack surface of a cloud resource.

For example, opening SSH, HTTP, database, or administrative ports to broad source ranges could create unnecessary exposure.

This alert complements the removal alert by monitoring both sides of the security group change lifecycle:

- Ingress rules being added
- Ingress rules being removed

---

### Validation

I validated this alert by adding a controlled temporary ingress rule to the EC2 instance’s attached security group.

Validation confirmed that:

- CloudTrail recorded the `AuthorizeSecurityGroupIngress` event
- The CloudWatch Logs metric filter matched the event
- The custom metric `AuthorizeSecurityGroupIngressEvents` received a datapoint
- The alarm entered the `ALARM` state
- CloudWatch published the alarm notification to SNS
- SNS delivered the email notification successfully

---

### Operational Meaning

This alert indicates that an ingress rule was added to an AWS security group.

From an analyst perspective, this could represent:

- Legitimate administrative maintenance
- Intentional service exposure
- Accidental misconfiguration
- Unauthorized expansion of network exposure

Follow-up investigation could include:

- Confirming who made the change
- Reviewing the port, protocol, and source range added
- Determining whether the change was expected or approved
- Checking what resource the security group is attached to
- Evaluating whether the new rule introduced unnecessary exposure

---

## Alert 4: IAM Managed Policy Attachment

### Alerting Goal

This alert detects when an IAM managed policy is attached directly to a user.

It is based on AWS identity-control-plane activity captured by CloudTrail.

---

### Metric Used

```text
CloudSecurityMonitoring / AttachUserPolicyEvents
```

This metric counts CloudTrail events where a managed IAM policy is attached directly to a user.

---

### Alarm Configuration

```text
Alarm name: attach-user-policy-alarm
Metric: AttachUserPolicyEvents
Namespace: CloudSecurityMonitoring
Threshold: Greater than or equal to 1
Evaluation window: 1 datapoint within 5 minutes
Alarm action: Publish to SNS topic cloud-security-monitoring-alerts
```

A single `AttachUserPolicy` event within the evaluation period is enough to trigger the alarm.

---

### Why This Alert Matters

IAM policy attachment can expand the permissions available to an identity.

Unexpected policy attachment activity may indicate:

- Unauthorized access expansion
- Privilege escalation
- Risky administrative activity
- Post-compromise permission changes
- Direct attachment of sensitive managed policies

The lab used `IAMReadOnlyAccess` for safe validation, but the same alerting logic would also detect higher-impact policy attachments.

---

### Validation

I validated this alert by attaching the AWS managed `IAMReadOnlyAccess` policy to a test IAM user.

Test user:

```text
attach-policy-test-user
```

Validation confirmed that:

- CloudTrail recorded the `AttachUserPolicy` event
- The CloudWatch Logs metric filter matched the event
- The custom metric `AttachUserPolicyEvents` received a datapoint
- The alarm entered the `ALARM` state
- CloudWatch published the alarm notification to SNS
- SNS delivered the email notification successfully

---

### Operational Meaning

This alert indicates that a managed IAM policy was attached directly to a user.

From an analyst perspective, this could represent:

- Legitimate IAM administration
- Unauthorized permission expansion
- Privilege escalation
- Attacker access expansion after credential compromise
- Persistence or preparation for broader AWS access

Follow-up investigation could include:

- Confirming who attached the policy
- Reviewing which user received the policy
- Reviewing which policy was attached
- Determining whether the change was expected or approved
- Checking for other IAM activity before and after the event
- Reviewing source IP, session context, and access-key characteristics

---

## Validation Summary

| Alert Path | Event Generated | Metric Updated | Alarm Triggered | SNS Delivered |
|---|---:|---:|---:|---:|
| Invalid SSH user attempts | Yes | Yes | Yes | Yes |
| Security group ingress rule removal | Yes | Yes | Yes | Yes |
| Security group ingress rule addition | Yes | Yes | Yes | Yes |
| IAM managed policy attachment | Yes | Yes | Yes | Yes |

---

## Troubleshooting During Validation

One useful part of this phase was troubleshooting notification delivery.

During earlier host-side validation, the original Gmail-based SNS email subscription repeatedly became deactivated shortly after confirmation. This appeared to be an email endpoint or link-handling issue rather than a CloudWatch alarm issue.

To isolate the problem, I moved the SNS subscription to a different email endpoint and repeated the test successfully.

This troubleshooting helped isolate the failure domain:

| Component | Status |
|---|---|
| Host telemetry | Working |
| CloudWatch Logs ingestion | Working |
| Metric filter | Working |
| CloudWatch alarm | Working |
| Original email endpoint path | Unstable |
| Alternate email endpoint | Successful |

This reinforced that alert delivery paths need to be validated just as carefully as detection logic.

---

## Evidence

### Host-Side CloudWatch Alarm Trigger

![CloudWatch alarm entering the ALARM state after invalid-user SSH attempts exceeded threshold](../screenshots/cloudwatch-alarm-invalid-user-triggered.png)

### Host-Side SNS Email Alert Delivery

![Email notification showing successful delivery of the invalid-user SSH alarm](../screenshots/sns-email-invalid-user-alarm-icloud.png.jpeg)

### CloudTrail Revoke CloudWatch Alarm Trigger

![CloudWatch alarm entering the ALARM state after a RevokeSecurityGroupIngress event was detected](../screenshots/cloudwatch-alarm-revoke-security-group-ingress-triggered.png)

### CloudTrail Revoke SNS Email Alert Delivery

![Email notification showing successful delivery of the RevokeSecurityGroupIngress alarm](../screenshots/sns-email-revoke-security-group-ingress-alarm.png)

### CloudTrail Authorize CloudWatch Alarm Trigger

![CloudWatch alarm entering the ALARM state after an AuthorizeSecurityGroupIngress event was detected](../screenshots/cloudwatch-alarm-authorize-security-group-ingress-triggered.png)

### CloudTrail Authorize SNS Email Alert Delivery

![Email notification showing successful delivery of the AuthorizeSecurityGroupIngress alarm](../screenshots/sns-email-authorize-security-group-ingress-alarm.png)

### IAM AttachUserPolicy CloudWatch Alarm Trigger

![CloudWatch alarm entering the ALARM state after an AttachUserPolicy event was detected](../screenshots/cloudwatch-alarm-attach-user-policy-triggered.png)

### IAM AttachUserPolicy SNS Email Alert Delivery

![Email notification showing successful delivery of the AttachUserPolicy alarm](../screenshots/sns-email-attach-user-policy-alarm.png)

---

## Limitations

This alerting workflow is functional but intentionally simple.

Current limitations include:

- Alerting is email-based only
- There is no automated containment or remediation action
- The host-side alarm is based on one SSH-related behavior
- The cloud-side alarms are based on selected CloudTrail event types
- The workflow does not correlate host-side and cloud-side events
- Thresholds are intentionally simple for the initial lab implementation
- IAM alerting is limited to one event type in this version of the lab

These limitations are acceptable for this lab because the goal was to prove the full alerting path from telemetry to notification.

---

## Key Takeaway

This phase completed four end-to-end alerting paths.

On the host side, repeated invalid-user SSH attempts were converted into the `InvalidUserSSHAttempts` metric, evaluated by a CloudWatch alarm, and delivered through SNS.

On the cloud network-control-plane side, AWS security group ingress rule removal and addition events were converted into the `RevokeSecurityGroupIngressEvents` and `AuthorizeSecurityGroupIngressEvents` metrics, evaluated by CloudWatch alarms, and delivered through SNS.

On the cloud identity-control-plane side, IAM managed policy attachment events were converted into the `AttachUserPolicyEvents` metric, evaluated by a CloudWatch alarm, and delivered through SNS.

Together, these alerting paths show that the lab can move from host telemetry and AWS administrative activity to centralized detection, alarm state changes, and actionable notification delivery.
