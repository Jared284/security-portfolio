# 07 - Reflections and Improvements

## Current Reflection

This lab is now much stronger than a basic AWS logging demo because it moved beyond setup and into real validation across multiple telemetry sources and multiple cloud control-plane detections.

At this stage, I have successfully built and validated both:

- a **host-side monitoring path** based on Linux authentication logs from an EC2 instance
- a **cloud-side monitoring path** based on AWS control-plane activity captured through CloudTrail

That matters because the lab is not limited to one narrow signal type. Instead, it now demonstrates monitoring across both workload-level and administrative activity in AWS.

## What This Lab Does Well So Far

The strongest part of the lab right now is that it contains multiple complete monitoring pipelines.

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

This is valuable because it shows that the lab can monitor security-relevant AWS administrative actions that reduce or change network exposure.

### 3. CloudTrail-Side Signal Path: Ingress Rule Addition

The second cloud-side path demonstrates:

1. a controlled security group rule addition was made in AWS
2. CloudTrail recorded the resulting `AuthorizeSecurityGroupIngress` event
3. the event was available in CloudWatch Logs
4. a metric filter converted that event into a custom metric
5. a CloudWatch alarm evaluated the event against a threshold
6. Amazon SNS delivered the resulting alert by email

This is valuable because it shows that the lab can also monitor when network exposure is opened, not just when it is reduced or removed.

### Why That Matters

Together, these paths make the project more credible because they demonstrate:

- multi-source telemetry collection
- centralized monitoring in AWS
- basic cloud-native detection engineering
- threshold-based alerting
- end-to-end validation of security controls
- monitoring of both workload activity and AWS control-plane changes

The project also benefits from being organized into separate sections for architecture, build, ingestion, attack simulation, detection engineering, alerting, and reflections. That structure makes it easier to explain and defend.

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

A separate lesson came from attempting to use failed console login activity as the next cloud-side detection. While `ConsoleLogin` events were visible and the success state was easy to inspect, failed-login validation became messy enough that it was not the best use of time for this phase of the lab. Pivoting to paired security group change detections was the better decision because it produced faster, cleaner, and more defensible results.

## Current Limitations

Even though the lab is much stronger now, it is still not complete.

The main limitations right now are:

- the host-side detection path still centers on one main SSH-related behavior
- the cloud-side detection path currently centers on a small set of related event types: `AuthorizeSecurityGroupIngress` and `RevokeSecurityGroupIngress`
- alerting is currently email-based only
- there is no automated response or remediation action
- the host-side detection uses simple string matching rather than richer parsing
- the cloud-side detections use specific event filters rather than broader control-plane correlation
- the lab does not yet correlate host-level activity with AWS control-plane activity

These limitations do not weaken the validated work that already exists, but they do define what still needs to improve for the project to feel more mature.

## Most Important Next Improvements

The biggest next improvement is to expand the cloud-side detection coverage beyond security group ingress modifications.

That means implementing additional CloudTrail-based detections for AWS administrative activity, such as:

- failed AWS console login activity
- IAM policy, role, or user changes
- suspicious administrative API activity
- trail modification or disablement attempts

Beyond that, the lab could be improved further by:

- adding more than one host-based detection rule
- tightening thresholds and alerting logic to reduce noise
- adding automated response options
- improving screenshot consistency and diagram polish
- documenting analyst follow-up steps more deeply for each alert type
- clarifying how host-side and cloud-side detections complement each other

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

I would also want to strengthen the narrative around why different telemetry sources matter and how they can be used together to improve visibility.

## Key Takeaway

The most important outcome so far is that this lab now proves I can build and validate real monitoring paths in AWS across both:

- host-level Linux authentication telemetry
- AWS control-plane activity captured through CloudTrail

More specifically, the cloud-side coverage now includes detection and alerting for both security group ingress rule additions and removals, which makes the control-plane monitoring story more complete than a single one-off event.

That is a meaningful step up from a simple AWS setup exercise.

At the same time, the project is still unfinished, and that matters. The next phase is to expand the number and quality of detections so the final lab shows broader cloud detection engineering depth rather than just a small set of validated signal paths.
