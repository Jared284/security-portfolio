# 04 - Attack Simulation and Validation Activity

## Objective

The goal of this phase was to generate security-relevant activity that could be used to validate the lab’s monitoring and detection pipelines.

This included:

- repeated invalid-user SSH login attempts against the EC2 monitoring host
- controlled security group ingress rule changes
- controlled IAM managed policy attachment activity

These actions were used to confirm that the lab could capture real telemetry from both the host and AWS control plane, then use that telemetry for detection and alerting.

---

## Validation Strategy

The lab uses two categories of test activity.

### 1. Host-Side Attack Simulation

The host-side test simulated an external attacker attempting to access the EC2 instance over SSH using usernames that did not exist on the Linux host.

This generated `Invalid user` entries in:

```text
/var/log/auth.log
```

Those events were later forwarded into CloudWatch Logs and used for host-based detection.

### 2. CloudTrail-Side Controlled Validation

The cloud-side tests used controlled AWS administrative actions to generate CloudTrail events.

These actions were not destructive attacks. They were safe validation steps designed to produce specific AWS control-plane events for detection engineering.

The CloudTrail-side validation actions generated:

```text
RevokeSecurityGroupIngress
AuthorizeSecurityGroupIngress
AttachUserPolicy
```

These events were later used to validate CloudWatch metric filters, custom metrics, alarms, and SNS notifications.

---

## Host-Side Attack Scenario

I simulated an external attacker attempting to access the EC2 instance over SSH using usernames that did not exist on the target Linux host.

Instead of trying valid usernames with incorrect passwords, I intentionally used fake usernames to generate `Invalid user` log entries. This mattered because the detection logic for this phase was built around matching those specific authentication events.

### Target

- **Target host:** `cloud-security-monitoring-host`
- **Target type:** AWS EC2 Ubuntu instance
- **Exposed service:** SSH on port `22`
- **Public IP used during testing:** `44.201.91.146`

### Execution

From my local workstation, I generated repeated SSH login attempts using fake usernames:

```bash
ssh fakeadmin@44.201.91.146
ssh backupsvc@44.201.91.146
ssh tempops@44.201.91.146
ssh nobody123@44.201.91.146
ssh svc-test@44.201.91.146
```

Each attempt failed with a `Permission denied (publickey)` response, confirming that the requests reached the target host and were rejected by the SSH service.

### Host-Side Validation

After generating the SSH attempts, I confirmed that the EC2 instance recorded the activity in `/var/log/auth.log`.

Example log entries included:

```text
Invalid user backupsvc from 128.6.147.103
Invalid user tempops from 128.6.147.103
Invalid user nobody123 from 128.6.147.103
Invalid user svc-test from 128.6.147.103
```

This confirmed that the simulated SSH activity produced the exact host-level telemetry needed for detection engineering and alerting.

---

## CloudTrail-Side Validation Scenarios

The CloudTrail detections were validated by generating controlled AWS management events.

### Scenario 1 - Security Group Ingress Rule Removal

To validate detection of security group ingress rule removal, I removed an ingress rule from the EC2 instance’s attached security group.

This generated the CloudTrail event:

```text
RevokeSecurityGroupIngress
```

This event was later used to validate:

- CloudTrail event capture
- CloudWatch Logs ingestion
- metric filter matching
- custom metric creation
- CloudWatch alarm triggering
- SNS email notification delivery

### Scenario 2 - Security Group Ingress Rule Addition

To validate detection of security group ingress rule addition, I added an ingress rule to the EC2 instance’s attached security group.

This generated the CloudTrail event:

```text
AuthorizeSecurityGroupIngress
```

This event was later used to validate:

- CloudTrail event capture
- CloudWatch Logs ingestion
- metric filter matching
- custom metric creation
- CloudWatch alarm triggering
- SNS email notification delivery

### Scenario 3 - IAM Managed Policy Attachment

To validate detection of IAM managed policy attachment activity, I attached the AWS managed `IAMReadOnlyAccess` policy directly to a test IAM user.

The test user was:

```text
attach-policy-test-user
```

The policy used for validation was:

```text
IAMReadOnlyAccess
```

This generated the CloudTrail event:

```text
AttachUserPolicy
```

This event was later used to validate:

- CloudTrail event capture
- CloudWatch Logs ingestion
- metric filter matching
- custom metric creation
- CloudWatch alarm triggering
- SNS email notification delivery

---

## Why These Tests Were Chosen

These tests were chosen because they produce clear, repeatable security signals without requiring destructive activity or successful compromise.

The SSH simulation represents workload-level authentication activity.

The security group tests represent network-control-plane changes that could affect exposure of cloud resources.

The IAM policy attachment test represents identity-control-plane activity that could indicate privilege expansion if performed unexpectedly.

Together, these validation actions created a clear signal path across multiple layers:

- local test activity
- host or CloudTrail telemetry
- centralized CloudWatch Logs
- CloudWatch metric filters
- custom CloudWatch metrics
- CloudWatch alarms
- SNS notification delivery

---

## Evidence

### Local SSH Attack Simulation

![Repeated invalid-user SSH attempts from the local workstation](../screenshots/ssh-invalid-user-attempts-terminal.png)

### Host Log Validation

![EC2 auth.log entries showing invalid-user SSH attempts](../screenshots/auth-log-invalid-user-events.png)

### IAM Policy Attachment Validation

![IAM managed policy attached to test user](../screenshots/iam-user-policy-attachment-action.png)

### CloudTrail AttachUserPolicy Event

![CloudTrail Event history showing AttachUserPolicy](../screenshots/cloudtrail-attach-user-policy-event.png)

---

## Key Takeaway

This phase generated the activity needed to validate the lab’s monitoring pipeline.

On the host side, repeated invalid-user SSH attempts produced real Linux authentication events.

On the cloud side, controlled security group and IAM actions produced CloudTrail management events.

Together, these tests confirmed that the lab could generate meaningful telemetry across workload-level activity, network-control-plane changes, and identity-control-plane changes.
