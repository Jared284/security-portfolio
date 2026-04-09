# 07 - Reflections and Improvements

## Current Reflection

This lab has already become much stronger than a basic AWS logging demo because it moved beyond setup and into real validation.

At this stage, I have successfully built and validated the host-side monitoring path of the project. That includes:

- forwarding Linux authentication logs from an EC2 instance into CloudWatch Logs
- creating a custom metric from `Invalid user` SSH events
- configuring a CloudWatch alarm on that metric
- routing the alarm through Amazon SNS
- validating end-to-end email delivery after troubleshooting the notification path

That validation work mattered because it proved the pipeline was functioning across multiple layers rather than just existing on paper.

## What This Lab Does Well So Far

The strongest part of the lab right now is the full host-based signal path:

1. external SSH activity was simulated from a local workstation
2. the EC2 instance recorded the activity in `/var/log/auth.log`
3. the CloudWatch Agent forwarded that telemetry into CloudWatch Logs
4. a metric filter converted matching log patterns into a custom metric
5. a CloudWatch alarm evaluated the threshold condition
6. Amazon SNS delivered the resulting alert by email

This is valuable because it shows the difference between merely configuring cloud services and actually validating a security monitoring workflow end to end.

The lab also benefits from being organized into separate sections for architecture, build, ingestion, attack simulation, detection engineering, and alerting. That structure makes the project easier to explain and defend.

## Challenges Encountered

One of the most useful parts of this lab was the troubleshooting process.

The original Gmail-based SNS subscription repeatedly became deactivated shortly after confirmation. Based on testing, that issue appeared to be related to the email endpoint or link-handling path rather than the AWS alarming logic itself.

That troubleshooting process helped isolate the problem correctly:

- the host logs were working
- CloudWatch log ingestion was working
- the metric filter was working
- the CloudWatch alarm was working
- the original email endpoint path was unstable
- switching to a different email endpoint allowed end-to-end delivery validation to complete successfully

This was a useful reminder that alert delivery paths need to be validated just as carefully as the detection logic itself.

## Current Limitations

Although the host-side path is validated, the overall lab is still incomplete.

The main limitations right now are:

- only one primary host-based detection has been fully engineered and validated
- the CloudTrail-side detection path has not yet been built out to the same level
- alerting is currently email-based only
- there is no automated response or remediation action
- the implemented detection uses simple string matching rather than deeper log parsing or event correlation
- the lab does not yet correlate host-level activity with AWS control-plane activity

These limitations do not weaken the completed host-side work, but they do define what still needs to be improved for the lab to feel fully mature.

## Most Important Next Improvements

The highest-priority next step is to build the CloudTrail-side detection path so the lab fully reflects both telemetry sources described in the project scope.

That means implementing one or more detections for AWS control-plane activity, such as:

- security group modifications
- failed console login activity
- IAM policy, role, or user changes
- suspicious administrative API activity

Beyond that, the lab could be improved further by:

- adding more than one host-based detection rule
- tightening threshold logic to reduce noise
- adding automated response options
- improving screenshot consistency and diagram polish
- documenting analyst follow-up steps more deeply for each alert type

## What I Would Improve in a Future Version

In a more advanced version of this lab, I would want to push it beyond basic CloudWatch-native monitoring and make it feel closer to a lightweight cloud detection engineering project.

Future improvements could include:

- richer CloudTrail detections
- multiple host-based detections instead of only one
- correlation between host telemetry and cloud administrative events
- automated remediation or containment actions
- clearer severity tiers for different alert types
- more formal investigation playbooks for each detection

I would also want to tighten the architecture narrative so the project more clearly shows the relationship between host visibility and cloud control-plane visibility.

## Key Takeaway

The most important outcome so far is that this lab already proves I can build and validate a real monitoring path in AWS from raw host telemetry to actionable alert delivery.

At the same time, the project is not finished, and that matters. The next phase is to bring the CloudTrail side of the lab up to the same standard so the final project shows both host-level and cloud-level security monitoring depth.
