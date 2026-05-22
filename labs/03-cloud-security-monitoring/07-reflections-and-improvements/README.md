# 07 - Reflections and Improvements

## Purpose

This section reflects on what the AWS Cloud Security Monitoring Lab successfully demonstrated, what challenges came up during the build, what limitations remain, and how the project could be improved in a future version.

The goal of this section is to show that the lab was not only built, but also evaluated from a security engineering perspective.

---

## Current Reflection

This lab became stronger than a basic AWS logging demo because it moved beyond setup and into validated detection and alerting across multiple telemetry sources.

The final lab includes:

- A **host-side monitoring path** based on Linux authentication logs from an EC2 instance
- A **cloud network-control-plane monitoring path** based on AWS security group changes captured through CloudTrail
- A **cloud identity-control-plane monitoring path** based on IAM policy attachment activity captured through CloudTrail

That matters because the lab is not limited to one narrow signal type. It demonstrates monitoring across workload-level activity, AWS network administration, and AWS identity administration.

---

## What This Lab Does Well

The strongest part of the lab is that it contains multiple complete monitoring pipelines.

Each pipeline follows the same general structure:

```text
Controlled activity
        ↓
Telemetry generation
        ↓
Centralized log ingestion
        ↓
Metric filter matching
        ↓
Custom metric creation
        ↓
CloudWatch alarm evaluation
        ↓
SNS notification delivery
```

This makes the lab more credible because it proves the detections were not just theoretical. They were validated end to end.

---

## Completed Monitoring Paths

## 1. Host-Side Signal Path

The host-side path demonstrates:

1. External SSH activity was simulated from a local workstation.
2. The EC2 instance recorded the activity in `/var/log/auth.log`.
3. The CloudWatch Agent forwarded that telemetry into CloudWatch Logs.
4. A metric filter converted matching log patterns into a custom metric.
5. A CloudWatch alarm evaluated the threshold condition.
6. Amazon SNS delivered the resulting alert by email.

This is valuable because it shows how host-level authentication telemetry can be turned into a real detection and alerting workflow.

---

## 2. CloudTrail Signal Path: Ingress Rule Removal

The first cloud-side path demonstrates:

1. A controlled security group rule removal was made in AWS.
2. CloudTrail recorded the resulting `RevokeSecurityGroupIngress` event.
3. The event was available in CloudWatch Logs.
4. A metric filter converted that event into a custom metric.
5. A CloudWatch alarm evaluated the event against a threshold.
6. Amazon SNS delivered the resulting alert by email.

This is valuable because it shows that the lab can monitor AWS administrative actions that modify network access controls.

---

## 3. CloudTrail Signal Path: Ingress Rule Addition

The second cloud-side path demonstrates:

1. A controlled security group rule addition was made in AWS.
2. CloudTrail recorded the resulting `AuthorizeSecurityGroupIngress` event.
3. The event was available in CloudWatch Logs.
4. A metric filter converted that event into a custom metric.
5. A CloudWatch alarm evaluated the event against a threshold.
6. Amazon SNS delivered the resulting alert by email.

This is valuable because it shows that the lab can monitor when network exposure is opened, not just when access rules are removed.

---

## 4. CloudTrail Signal Path: IAM Managed Policy Attachment

The IAM path demonstrates:

1. A managed IAM policy was attached to a test IAM user.
2. CloudTrail recorded the resulting `AttachUserPolicy` event.
3. The event was available in CloudWatch Logs.
4. A metric filter converted the event into a custom metric.
5. A CloudWatch alarm evaluated the event against a threshold.
6. Amazon SNS delivered the resulting alert by email.

This is valuable because IAM changes are one of the most important areas of AWS security monitoring. A policy attachment can represent normal administration, but it can also indicate privilege expansion if performed unexpectedly.

---

## Skills Demonstrated

This lab demonstrates practical skills across cloud security monitoring and detection engineering, including:

- AWS CloudTrail configuration and event validation
- CloudWatch Logs ingestion
- CloudWatch metric filter design
- Custom CloudWatch metric creation
- CloudWatch alarm configuration
- SNS alert delivery
- EC2 host log forwarding with the CloudWatch Agent
- Linux authentication log review
- AWS control-plane monitoring
- IAM activity monitoring
- Security group change monitoring
- End-to-end detection validation
- Alert delivery troubleshooting

---

## Why This Lab Matters

The most important strength of this lab is that it connects multiple layers of security visibility.

It monitors:

- Activity inside a Linux workload
- AWS network-control-plane changes
- AWS identity-control-plane changes

That distinction matters because real cloud security monitoring cannot rely on only one telemetry source.

A defender needs visibility into both workload behavior and AWS account-level administrative activity.

This lab demonstrates that relationship in a focused and defensible way.

---

## Challenges Encountered

One of the most useful parts of this lab was the troubleshooting process.

During earlier host-side validation, the original Gmail-based SNS email subscription repeatedly became deactivated shortly after confirmation. Based on testing, that issue appeared to be related to the email endpoint or link-handling path rather than the AWS alarming logic itself.

That troubleshooting process helped isolate the problem correctly:

| Component | Status |
|---|---|
| Host logs | Working |
| CloudWatch Logs ingestion | Working |
| Metric filters | Working |
| CloudWatch alarms | Working |
| Original email endpoint path | Unstable |
| Alternate email endpoint | Successful |

This was a useful reminder that alert delivery paths need to be validated just as carefully as detection logic.

---

## Lessons Learned

### 1. CloudTrail Event Names Should Be Validated Directly

Working with CloudTrail reinforced the importance of confirming exact event names before building detection rules.

CloudTrail Event history was useful for confirming event names such as:

```text
RevokeSecurityGroupIngress
AuthorizeSecurityGroupIngress
AttachUserPolicy
```

This helped avoid guessing and made the metric filters more defensible.

---

### 2. Detection Quality Depends on Reliable Ingestion

The detection logic only mattered because the ingestion layer worked.

The host-side path depended on the CloudWatch Agent forwarding `/var/log/auth.log`.

The cloud-side path depended on CloudTrail delivering events into CloudWatch Logs.

This reinforced that log ingestion is not just setup work. It is part of the security control.

---

### 3. Alert Delivery Must Be Tested End to End

A detection is incomplete if the alert never reaches the expected destination.

The SNS troubleshooting showed that even when logs, metric filters, and alarms are working, the notification path can still fail.

That is why the final validation included actual email delivery, not just alarm state changes.

---

### 4. Metric Graphs Need the Right Statistic

For event-count detections, using the `Sum` statistic was important.

Other metric views can make the graph look misleading or empty even when the metric filter is working.

This reinforced the importance of understanding how CloudWatch displays metric data.

---

### 5. Scope Control Matters

I initially considered failed AWS console login detection as another CloudTrail use case.

While `ConsoleLogin` events were visible, failed-login validation became less clean than the other paths. Pivoting to security group changes and IAM policy attachment produced clearer, more defensible detection paths for this version of the lab.

That was the better project decision.

---

## Current Limitations

The core lab is complete, but it is intentionally small.

Current limitations include:

- The host-side detection path centers on one SSH-related behavior
- The cloud-side detection path covers a small set of selected CloudTrail events
- IAM detection coverage is limited to `AttachUserPolicy`
- Alerting is email-based only
- There is no automated response or remediation action
- The host-side detection uses simple string matching
- The CloudTrail-side detections use specific event filters rather than broader correlation
- Host-level activity and AWS control-plane activity are not correlated together
- The lab is CloudWatch-native and does not use a full SIEM

These limitations do not weaken the validated work. They define the boundary of this version of the lab and point toward future improvements.

---

## Future Improvements

The biggest future improvement would be expanding CloudTrail-side detection coverage beyond the current validated signals.

Strong future candidates include:

- Failed AWS console login activity
- `CreateAccessKey` events
- `PutUserPolicy` events
- `AttachRolePolicy` events
- CloudTrail modification or disablement attempts
- Suspicious administrative API activity
- Unusual security group changes
- Root account activity
- Unauthorized region activity

---

## Advanced Future Improvements

In a more advanced version of this lab, I would want to push it beyond basic CloudWatch-native monitoring and make it closer to a lightweight cloud detection engineering project.

Future improvements could include:

- Multiple host-based detections instead of one SSH-focused rule
- Correlation between host telemetry and cloud administrative events
- Automated remediation or containment actions
- Severity tiers for different alert types
- Investigation playbooks for each detection
- Better architectural visualization of monitoring paths
- MITRE ATT&CK mapping for detection logic
- Integration with a SIEM such as Splunk, Elastic, or Microsoft Sentinel
- Infrastructure as Code deployment using Terraform or CloudFormation

---

## Final Takeaway

The most important outcome is that this lab proves I can build and validate real monitoring paths in AWS across:

- Host-level Linux authentication telemetry
- AWS network-control-plane activity captured through CloudTrail
- AWS identity-control-plane activity captured through CloudTrail

More specifically, the lab includes detection and alerting for:

- Repeated invalid-user SSH attempts
- Security group ingress rule removals
- Security group ingress rule additions
- IAM managed policy attachments

That is a meaningful step up from a simple AWS setup exercise.

The project is still small compared with a production SIEM or mature cloud detection program, but the core workflow is real:

```text
Generate activity
        ↓
Capture telemetry
        ↓
Create detection logic
        ↓
Trigger alarm
        ↓
Validate notification delivery
```

That workflow is the foundation of practical cloud detection engineering.
