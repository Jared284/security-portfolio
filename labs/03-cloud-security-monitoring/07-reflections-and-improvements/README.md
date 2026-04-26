# 07 - Reflections and Improvements

## Current Reflection

This lab is much stronger than a basic AWS logging demo because it moved beyond setup and into validated detection and alerting across multiple telemetry sources.

At this stage, I successfully built and validated:

- a **host-side monitoring path** based on Linux authentication logs from an EC2 instance
- a **cloud network-control-plane monitoring path** based on AWS security group changes captured through CloudTrail
- a **cloud identity-control-plane monitoring path** based on IAM policy attachment activity captured through CloudTrail

That matters because the lab is not limited to one narrow signal type. It demonstrates monitoring across workload-level activity, AWS network administration, and AWS identity administration.

---

## What This Lab Does Well

The strongest part of the lab is that it contains multiple complete monitoring pipelines.

### 1. Host-Side Signal Path

The host-side path demonstrates:

1. external SSH activity was simulated from a local workstation
2. the EC2 instance recorded the activity in `/var/log/auth.log`
3. the CloudWatch Agent forwarded that telemetry into CloudWatch Logs
4. a metric filter converted matching log patterns into a custom metric
5. a CloudWatch alarm evaluated the threshold condition
6. Amazon SNS delivered the resulting alert by email

This is valuable because it shows how host-level authentication telemetry can be turned into a real detection and alerting workflow.

### 2. CloudTrail-Side Signal Path: Ingress Rule Removal

The first cloud-side path demonstrates:

1. a controlled security group rule removal was made in AWS
2. CloudTrail recorded the resulting `RevokeSecurityGroupIngress` event
3. the event was available in CloudWatch Logs
4. a metric filter converted that event into a custom metric
5. a CloudWatch alarm evaluated the event against a threshold
6. Amazon SNS delivered the resulting alert by email

This is valuable because it shows that the lab can monitor AWS administrative actions that modify network access controls.

### 3. CloudTrail-Side Signal Path: Ingress Rule Addition

The second cloud-side path demonstrates:

1. a controlled security group rule addition was made in AWS
2. CloudTrail recorded the resulting `AuthorizeSecurityGroupIngress` event
3. the event was available in CloudWatch Logs
4. a metric filter converted that event into a custom metric
5. a CloudWatch alarm evaluated the event against a threshold
6. Amazon SNS delivered the resulting alert by email

This is valuable because it shows that the lab can monitor when network exposure is opened, not just when access rules are removed.

### 4. CloudTrail-Side Signal Path: IAM Managed Policy Attachment

The IAM path demonstrates:

1. a managed IAM policy was attached to a test IAM user
2. CloudTrail recorded the resulting `AttachUserPolicy` event
3. the event was available for CloudWatch-based detection
4. a metric filter converted the event into a custom metric
5. a CloudWatch alarm evaluated the event against a threshold
6. Amazon SNS delivered the resulting alert by email

This is valuable because IAM changes are one of the most important areas of AWS security monitoring. A policy attachment can represent normal administration, but it can also indicate privilege expansion if performed unexpectedly.

### Why That Matters

Together, these paths make the project more credible because they demonstrate:

- multi-source telemetry collection
- centralized monitoring in AWS
- host-based detection engineering
- CloudTrail-based control-plane detection engineering
- monitoring for network exposure changes
- monitoring for IAM permission changes
- threshold-based alerting
- end-to-end validation of security controls

The project also benefits from being organized into separate sections for architecture, build, ingestion, validation activity, detection engineering, alerting, and reflections. That structure makes it easier to explain and defend.

---

## Challenges Encountered

One of the most useful parts of this lab was the troubleshooting process.

The original Gmail-based SNS subscription repeatedly became deactivated shortly after confirmation. Based on testing, that issue appeared to be related to the email endpoint or link-handling path rather than the AWS alarming logic itself.

That troubleshooting process helped isolate the problem correctly:

- the host logs were working
- CloudWatch log ingestion was working
- the metric filters were working
- the CloudWatch alarms were working
- the original email endpoint path was unstable
- switching to a different email endpoint allowed end-to-end delivery validation to complete successfully

This was a useful reminder that alert delivery paths need to be validated just as carefully as the detection logic itself.

Another useful lesson came from working with CloudTrail and CloudWatch Logs. The log search workflow was not always the fastest way to identify the correct control-plane event, and CloudTrail Event history was a more effective way to confirm the exact AWS administrative event name before building the detection rule.

That mattered because it reinforced the importance of validating the event source and event name directly rather than guessing and building a filter blindly.

A separate lesson came from attempting to use failed console login activity as the next cloud-side detection. While `ConsoleLogin` events were visible and the success state was easy to inspect, failed-login validation became messy enough that it was not the best use of time for this phase of the lab. Pivoting to security group changes and IAM policy attachment produced cleaner, more defensible detection paths.

Another important lesson was that CloudWatch metric graphs need to be viewed carefully. For event-count detections, using the `Sum` statistic is important because other views can make the graph look misleading or empty even when the metric filter is working.

---

## Current Limitations

The core lab is complete, but it is still intentionally small.

The main limitations are:

- the host-side detection path centers on one SSH-related behavior
- the cloud-side detection path covers a small set of selected CloudTrail events
- the IAM detection coverage is limited to `AttachUserPolicy`
- alerting is currently email-based only
- there is no automated response or remediation action
- the host-side detection uses simple string matching rather than richer parsing
- the CloudTrail-side detections use specific event filters rather than broader control-plane correlation
- the lab does not yet correlate host-level activity with AWS control-plane activity

These limitations do not weaken the validated work. They define the boundary of this version of the lab and point toward future improvements.

---

## Most Important Future Improvements

The biggest future improvement would be to expand CloudTrail-side detection coverage beyond the current validated signals.

Strong future candidates include:

- failed AWS console login activity
- `CreateAccessKey` events
- `PutUserPolicy` events
- `AttachRolePolicy` events
- CloudTrail modification or disablement attempts
- suspicious administrative API activity

Beyond that, the lab could be improved further by:

- adding more than one host-based detection rule
- tightening thresholds and alerting logic to reduce noise
- adding automated response options
- improving screenshot consistency and diagram polish
- documenting analyst follow-up steps more deeply for each alert type
- clarifying how host-side and cloud-side detections complement each other

---

## What I Would Improve in a Future Version

In a more advanced version of this lab, I would want to push it beyond basic CloudWatch-native monitoring and make it feel closer to a lightweight cloud detection engineering project.

Future improvements could include:

- richer CloudTrail detections across more AWS services
- multiple host-based detections instead of only one SSH-focused rule
- correlation between host telemetry and cloud administrative events
- automated remediation or containment actions
- clearer severity tiers for different alert types
- more formal investigation playbooks for each detection
- better architectural visualization of the monitoring paths
- optional MITRE ATT&CK mapping for the detection logic

I would also want to strengthen the narrative around why different telemetry sources matter and how they can be used together to improve visibility.

---

## Key Takeaway

The most important outcome is that this lab proves I can build and validate real monitoring paths in AWS across:

- host-level Linux authentication telemetry
- AWS network-control-plane activity captured through CloudTrail
- AWS identity-control-plane activity captured through CloudTrail

More specifically, the lab now includes detection and alerting for:

- repeated invalid-user SSH attempts
- security group ingress rule removals
- security group ingress rule additions
- IAM managed policy attachments

That is a meaningful step up from a simple AWS setup exercise.

The project is still small compared with a production SIEM or mature cloud detection program, but the core workflow is real: generate activity, capture telemetry, create detection logic, trigger an alarm, and validate notification delivery.
