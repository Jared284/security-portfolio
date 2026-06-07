# 05 - Detection Engineering

## Purpose

This section documents how raw host and AWS control-plane telemetry was converted into measurable detection signals.

The goal of this phase was to move beyond log collection and prove that security-relevant events could be:

- Matched in CloudWatch Logs
- Converted into custom CloudWatch metrics
- Validated using controlled activity
- Prepared for CloudWatch alarm-based alerting

This phase is the core detection engineering layer of the lab.

---

## Detection Scope

This lab implements four detection paths.

| Detection | Telemetry Source | Event / Pattern | Detection Output |
|---|---|---|---|
| Invalid SSH user attempts | EC2 `/var/log/auth.log` | `Invalid user` | `InvalidUserSSHAttempts` |
| Security group ingress rule removal | CloudTrail | `RevokeSecurityGroupIngress` | `RevokeSecurityGroupIngressEvents` |
| Security group ingress rule addition | CloudTrail | `AuthorizeSecurityGroupIngress` | `AuthorizeSecurityGroupIngressEvents` |
| IAM managed policy attachment | CloudTrail | `AttachUserPolicy` | `AttachUserPolicyEvents` |

Together, these detections demonstrate visibility across:

- Host-level Linux authentication activity
- AWS network-control-plane changes
- AWS identity-control-plane changes

---

## Detection Engineering Workflow

Each detection followed the same workflow:

```text
Generate controlled activity
        ↓
Confirm raw log/event exists
        ↓
Create CloudWatch Logs metric filter
        ↓
Publish custom CloudWatch metric
        ↓
Validate metric datapoint
        ↓
Use metric for alarm configuration
```

This workflow shows the full path from raw security telemetry to an alert-ready detection signal.

---

## Shared Metric Namespace

All custom metrics were created under the same CloudWatch namespace:

```text
CloudSecurityMonitoring
```

Using one namespace kept the lab’s detection metrics organized and easy to review.

---

## Detection 1: Invalid SSH User Attempts

### Detection Goal

This detection identifies repeated invalid-user SSH login attempts against the EC2 monitoring host.

The goal was to detect suspicious authentication activity at the host level using Linux authentication logs.

---

### Telemetry Source

The telemetry source for this detection was:

```text
/var/log/auth.log
```

These logs were forwarded from the EC2 instance into CloudWatch Logs using the CloudWatch Agent.

CloudWatch log group:

```text
cloud-security-monitoring-auth
```

---

### Event Pattern

The detection focused on the following log pattern:

```text
Invalid user
```

Example observed entries:

```text
Invalid user backupsvc from 128.6.147.103
Invalid user tempops from 128.6.147.103
Invalid user nobody123 from 128.6.147.103
Invalid user svc-test from 128.6.147.103
```

---

### Metric Filter Configuration

```text
Filter name: invalid-user-ssh-attempts
Filter pattern: "Invalid user"
Metric namespace: CloudSecurityMonitoring
Metric name: InvalidUserSSHAttempts
Metric value: 1
```

Every matching `Invalid user` log entry increments the custom metric by `1`.

---

### Validation

I validated this detection by generating repeated fake SSH login attempts against the EC2 instance.

Validation confirmed that:

- The SSH attempts reached the EC2 instance
- The host wrote `Invalid user` entries to `/var/log/auth.log`
- The CloudWatch Agent forwarded those logs to CloudWatch Logs
- The metric filter matched the intended log pattern
- The custom metric `InvalidUserSSHAttempts` received datapoints

During validation, the metric showed a `Sum` of `5`, confirming that the simulated invalid-user SSH attempts were counted.

---

### Detection Output

```text
CloudSecurityMonitoring / InvalidUserSSHAttempts
```

This metric became the basis for the host-side CloudWatch alarm configured in the alerting phase.

---

## Detection 2: Security Group Ingress Rule Removal

### Detection Goal

This detection identifies when an ingress rule is removed from an AWS security group.

The goal was to monitor network-control-plane changes that modify cloud resource exposure.

---

### Telemetry Source

The telemetry source for this detection was AWS CloudTrail.

CloudTrail delivered the event into the CloudWatch log group:

```text
cloud-security-monitoring-cloudtrail
```

---

### Event Pattern

The detection focused on the following CloudTrail event:

```text
RevokeSecurityGroupIngress
```

Event source:

```text
ec2.amazonaws.com
```

---

### Metric Filter Configuration

```text
Filter name: revoke-security-group-ingress
Filter pattern: { ($.eventSource = "ec2.amazonaws.com") && ($.eventName = "RevokeSecurityGroupIngress") }
Metric namespace: CloudSecurityMonitoring
Metric name: RevokeSecurityGroupIngressEvents
Metric value: 1
```

Every matching `RevokeSecurityGroupIngress` CloudTrail event increments the custom metric by `1`.

---

### Why This Detection Matters

Security group ingress rules define what traffic can reach cloud resources.

Removing an ingress rule may be legitimate, but it is still a security-relevant control-plane event because it changes the network access posture of the environment.

This detection proves that AWS administrative activity can be converted into a measurable cloud-native security signal.

---

### Validation

I validated this detection by removing an ingress rule from the EC2 instance’s attached security group.

Validation confirmed that:

- CloudTrail recorded the `RevokeSecurityGroupIngress` event
- The event source was `ec2.amazonaws.com`
- The event was delivered to CloudWatch Logs
- The metric filter matched the intended CloudTrail event
- The custom metric `RevokeSecurityGroupIngressEvents` received a datapoint

During validation, the metric showed a datapoint of `1`.

---

### Detection Output

```text
CloudSecurityMonitoring / RevokeSecurityGroupIngressEvents
```

This metric became the basis for a CloudTrail-side CloudWatch alarm.

---

## Detection 3: Security Group Ingress Rule Addition

### Detection Goal

This detection identifies when an ingress rule is added to an AWS security group.

The goal was to monitor changes that may increase the exposure of a cloud resource.

---

### Telemetry Source

The telemetry source for this detection was AWS CloudTrail.

CloudTrail delivered the event into the CloudWatch log group:

```text
cloud-security-monitoring-cloudtrail
```

---

### Event Pattern

The detection focused on the following CloudTrail event:

```text
AuthorizeSecurityGroupIngress
```

Event source:

```text
ec2.amazonaws.com
```

---

### Metric Filter Configuration

```text
Filter name: authorize-security-group-ingress
Filter pattern: { ($.eventSource = "ec2.amazonaws.com") && ($.eventName = "AuthorizeSecurityGroupIngress") }
Metric namespace: CloudSecurityMonitoring
Metric name: AuthorizeSecurityGroupIngressEvents
Metric value: 1
```

Every matching `AuthorizeSecurityGroupIngress` CloudTrail event increments the custom metric by `1`.

---

### Why This Detection Matters

Security group ingress additions can increase the attack surface of a cloud resource.

For example, opening SSH, HTTP, database, or administrative ports to broad source ranges could create unnecessary exposure.

This detection complements the ingress rule removal detection by monitoring both sides of the security group change lifecycle:

- Ingress rules being added
- Ingress rules being removed

---

### Validation

I validated this detection by adding a controlled temporary ingress rule to the EC2 instance’s attached security group.

Validation confirmed that:

- CloudTrail recorded the `AuthorizeSecurityGroupIngress` event
- The event source was `ec2.amazonaws.com`
- The event was delivered to CloudWatch Logs
- The metric filter matched the intended CloudTrail event
- The custom metric `AuthorizeSecurityGroupIngressEvents` received a datapoint

During validation, the metric showed a datapoint of `1`.

---

### Detection Output

```text
CloudSecurityMonitoring / AuthorizeSecurityGroupIngressEvents
```

This metric became the basis for a CloudTrail-side CloudWatch alarm.

---

## Detection 4: IAM Managed Policy Attachment

### Detection Goal

This detection identifies when an IAM managed policy is attached directly to a user.

The goal was to monitor identity-control-plane activity that may indicate permission expansion.

---

### Telemetry Source

The telemetry source for this detection was AWS CloudTrail.

CloudTrail delivered the event into the CloudWatch log group:

```text
cloud-security-monitoring-cloudtrail
```

---

### Event Pattern

The detection focused on the following CloudTrail event:

```text
AttachUserPolicy
```

Event source:

```text
iam.amazonaws.com
```

---

### Metric Filter Configuration

```text
Filter name: attach-user-policy
Filter pattern: { ($.eventSource = "iam.amazonaws.com") && ($.eventName = "AttachUserPolicy") }
Metric namespace: CloudSecurityMonitoring
Metric name: AttachUserPolicyEvents
Metric value: 1
```

Every matching `AttachUserPolicy` CloudTrail event increments the custom metric by `1`.

---

### Why This Detection Matters

IAM policy attachment can expand the permissions available to an identity.

Unexpected policy attachment activity may indicate:

- Privilege escalation
- Unauthorized permission changes
- Misconfigured administrative activity
- Post-compromise access expansion

The validation used the lower-risk `IAMReadOnlyAccess` policy, but the same detection logic would also capture higher-impact managed policies such as `AdministratorAccess` or `PowerUserAccess`.

---

### Validation

I validated this detection by attaching the AWS managed `IAMReadOnlyAccess` policy directly to a test IAM user.

Test user:

```text
attach-policy-test-user
```

Validation confirmed that:

- CloudTrail recorded the `AttachUserPolicy` event
- The event source was `iam.amazonaws.com`
- The event included the expected IAM user and policy attachment activity
- The event was delivered to CloudWatch Logs
- The metric filter matched the intended CloudTrail event
- The custom metric `AttachUserPolicyEvents` received a datapoint

During validation, the metric showed a datapoint of `1`.

---

### Detection Output

```text
CloudSecurityMonitoring / AttachUserPolicyEvents
```

This metric became the basis for an IAM-focused CloudWatch alarm.

---

## Detection Summary

| Detection | Source | Filter / Event | Metric | Validation Result |
|---|---|---|---|---|
| Invalid SSH user attempts | `/var/log/auth.log` | `Invalid user` | `InvalidUserSSHAttempts` | Metric populated |
| Security group ingress rule removal | CloudTrail | `RevokeSecurityGroupIngress` | `RevokeSecurityGroupIngressEvents` | Metric populated |
| Security group ingress rule addition | CloudTrail | `AuthorizeSecurityGroupIngress` | `AuthorizeSecurityGroupIngressEvents` | Metric populated |
| IAM managed policy attachment | CloudTrail | `AttachUserPolicy` | `AttachUserPolicyEvents` | Metric populated |

---

## Why These Detection Paths Matter Together

The strongest part of this phase is that it does not rely on only one source of telemetry.

Instead, it demonstrates detection pipelines built from:

- Linux host authentication logs
- AWS CloudTrail network-control-plane events
- AWS CloudTrail identity-control-plane events

The final detection set shows monitoring coverage across:

- Workload-level activity
- Cloud administrative activity
- Network exposure changes
- IAM permission changes

This makes the lab stronger than a basic single-source logging project.

---

## Evidence

### Centralized Host Authentication Logs

![CloudWatch log entries showing forwarded invalid-user SSH events](../screenshots/cloudwatch-auth-log-invalid-user-events.png)

### Host-Side Metric Filter Configuration

![Metric filter configuration for invalid-user SSH detection](../screenshots/metric-filter-invalid-user-ssh-attempts.png)

### Host-Side Custom Metric Validation

![Custom CloudWatch metric showing summed invalid-user SSH attempts](../screenshots/cloudwatch-metric-invalid-user-ssh-attempts-sum-view.png)

### CloudTrail Revoke Security Group Change Event

![CloudTrail event showing RevokeSecurityGroupIngress for the monitored security group](../screenshots/cloudtrail-security-group-change-event.png)

### CloudTrail Revoke Metric Filter Configuration

![Metric filter configuration for RevokeSecurityGroupIngress detection](../screenshots/metric-filter-revoke-security-group-ingress.png)

### CloudTrail Revoke Custom Metric Validation

![Custom CloudWatch metric showing RevokeSecurityGroupIngress event detection](../screenshots/cloudwatch-metric-revoke-security-group-ingress-events.png)

### CloudTrail Authorize Security Group Change Event

![CloudTrail event showing AuthorizeSecurityGroupIngress for the monitored security group](../screenshots/cloudtrail-authorize-security-group-ingress-event.png)

### CloudTrail Authorize Custom Metric Validation

![Custom CloudWatch metric showing AuthorizeSecurityGroupIngress event detection](../screenshots/cloudwatch-metric-authorize-security-group-ingress-events.png)

### IAM Policy Attachment Action

![IAM managed policy attached directly to test user](../screenshots/iam-user-policy-attachment-action.png)

### CloudTrail AttachUserPolicy Event

![CloudTrail event showing AttachUserPolicy activity](../screenshots/cloudtrail-attach-user-policy-event.png)

### IAM Metric Filter Configuration

![Metric filter configuration for AttachUserPolicy detection](../screenshots/metric-filter-attach-user-policy.png.png)

### IAM Custom Metric Validation

![Custom CloudWatch metric showing AttachUserPolicy event detection](../screenshots/metric-attach-user-policy-datapoint.png.png)

---

## Limitations

These detections are intentionally focused and simple.

Current limitations include:

- Detection logic uses direct pattern or event-name matching
- Host-side detection does not perform deeper parsing or correlation
- CloudTrail-side detections focus on selected management events
- Host-level and CloudTrail telemetry are not correlated together
- Automated response is not implemented
- Coverage is limited to a small set of security-relevant events

These limitations are acceptable for this lab because the objective was to prove the full path from raw telemetry to custom detection metric.

---

## Key Takeaway

This phase turned multiple forms of raw telemetry into usable cloud-native detection signals.

On the host side, repeated invalid-user SSH activity was collected, matched, and converted into the `InvalidUserSSHAttempts` metric.

On the cloud side, AWS security group ingress rule additions, security group ingress rule removals, and IAM managed policy attachment events were recorded in CloudTrail, matched in CloudWatch Logs, and converted into custom CloudWatch metrics.

Together, these detection paths show that the lab can translate host-level, network-control-plane, and identity-control-plane events into actionable security signals ready for alerting.
