# Planning

## Purpose

This document tracks the working plan for the `cloud-security-monitoring` lab.

The goal of this lab is to build a centralized cloud security monitoring project in AWS that demonstrates visibility across both:

- host-level Linux authentication activity
- AWS control-plane activity

This file tracks progress, completed work, remaining polish, and future improvement ideas.

## Lab Scope

This project is designed to show that security-relevant telemetry from multiple sources can be centralized, monitored, and used for detection and alerting.

### Primary telemetry sources

1. **Host-level authentication logs**
   - EC2 Ubuntu instance
   - `/var/log/auth.log`
   - forwarded to CloudWatch Logs using the CloudWatch Agent

2. **AWS control-plane activity**
   - CloudTrail logs
   - delivered to both S3 and CloudWatch Logs
   - used to monitor AWS administrative activity, network-control-plane changes, and IAM permission changes

## Current Status

### Overall status

The core lab build is complete.

The lab now includes four validated detection and alerting paths:

1. invalid-user SSH attempts from EC2 host logs
2. `RevokeSecurityGroupIngress` CloudTrail events
3. `AuthorizeSecurityGroupIngress` CloudTrail events
4. `AttachUserPolicy` CloudTrail events

Each path has been validated from event generation through CloudWatch metric creation, alarm triggering, and SNS email notification delivery.

## Host-Side Monitoring Path

The host-side monitoring path is built and validated end to end.

Completed:

- EC2 Ubuntu monitoring host deployed
- IAM role created and attached to the EC2 instance
- `CloudWatchAgentServerPolicy` attached
- CloudWatch Agent installed and configured
- `/var/log/auth.log` forwarded into CloudWatch log group `cloud-security-monitoring-auth`
- repeated fake SSH login attempts simulated from local workstation
- `Invalid user` entries confirmed in local host logs
- `Invalid user` entries confirmed in CloudWatch Logs
- metric filter `invalid-user-ssh-attempts` created
- custom metric `CloudSecurityMonitoring / InvalidUserSSHAttempts` validated
- CloudWatch alarm `invalid-user-ssh-attempts-alarm` triggered successfully
- SNS email alert delivery validated

## CloudTrail-Side Monitoring Path

The CloudTrail-side monitoring path is built and validated across network-control-plane and IAM-control-plane events.

Completed:

- S3 bucket created for CloudTrail storage
- SNS topic created for alert delivery
- CloudTrail trail `cloud-security-monitoring-trail` created
- CloudTrail configured to deliver to S3
- CloudTrail configured to deliver to CloudWatch Logs
- CloudTrail telemetry validated in both S3 and CloudWatch
- CloudTrail log group `cloud-security-monitoring-cloudtrail` identified and validated
- controlled security group rule changes generated against the EC2 instance’s attached security group
- CloudTrail event `RevokeSecurityGroupIngress` identified in Event history
- CloudTrail event `AuthorizeSecurityGroupIngress` identified in Event history
- IAM managed policy attachment generated against a test IAM user
- CloudTrail event `AttachUserPolicy` identified in Event history
- metric filter `revoke-security-group-ingress` created
- metric filter `authorize-security-group-ingress` created
- metric filter `attach-user-policy` created
- custom metric `CloudSecurityMonitoring / RevokeSecurityGroupIngressEvents` validated
- custom metric `CloudSecurityMonitoring / AuthorizeSecurityGroupIngressEvents` validated
- custom metric `CloudSecurityMonitoring / AttachUserPolicyEvents` validated
- CloudWatch alarm `revoke-security-group-ingress-alarm` triggered successfully
- CloudWatch alarm `authorize-security-group-ingress-alarm` triggered successfully
- CloudWatch alarm `attach-user-policy-alarm` triggered successfully
- SNS email alert delivery validated for all CloudTrail-side detections

## Completed Detections

### 1. Invalid SSH User Attempts

- **Source:** EC2 `/var/log/auth.log`
- **Metric filter:** `invalid-user-ssh-attempts`
- **Metric:** `CloudSecurityMonitoring / InvalidUserSSHAttempts`
- **Alarm:** `invalid-user-ssh-attempts-alarm`
- **Purpose:** Detects repeated invalid-user SSH login attempts that may indicate username enumeration or brute-force activity.

### 2. Security Group Ingress Rule Removal

- **Source:** CloudTrail
- **Event name:** `RevokeSecurityGroupIngress`
- **Metric filter:** `revoke-security-group-ingress`
- **Metric:** `CloudSecurityMonitoring / RevokeSecurityGroupIngressEvents`
- **Alarm:** `revoke-security-group-ingress-alarm`
- **Purpose:** Detects security group ingress rule removal, which may indicate a meaningful network access control change.

### 3. Security Group Ingress Rule Addition

- **Source:** CloudTrail
- **Event name:** `AuthorizeSecurityGroupIngress`
- **Metric filter:** `authorize-security-group-ingress`
- **Metric:** `CloudSecurityMonitoring / AuthorizeSecurityGroupIngressEvents`
- **Alarm:** `authorize-security-group-ingress-alarm`
- **Purpose:** Detects security group ingress rule additions, which may indicate new network exposure.

### 4. IAM Managed Policy Attachment

- **Source:** CloudTrail
- **Event name:** `AttachUserPolicy`
- **Metric filter:** `attach-user-policy`
- **Metric:** `CloudSecurityMonitoring / AttachUserPolicyEvents`
- **Alarm:** `attach-user-policy-alarm`
- **Purpose:** Detects IAM managed policies being attached directly to users, which may indicate privilege expansion or unauthorized permission changes.

## Completed Documentation

The following sections are drafted or updated to reflect the completed host-side and CloudTrail-side work:

- root `README.md`
- `03-log-sources-and-ingestion/README.md`
- `04-attack-simulation/README.md`
- `05-detection-engineering/README.md`
- `06-alerting-and-response/README.md`
- `07-reflections-and-improvements/README.md`

The remaining documentation work is polish and consistency, not new AWS implementation.

## Remaining Work

The main remaining work is documentation cleanup.

### Documentation cleanup checklist

- confirm screenshot filenames match the names referenced in Markdown
- confirm every screenshot path renders correctly in GitHub
- remove duplicate or outdated screenshots
- make sure IAM detection is reflected consistently across the root README and section READMEs
- make sure the documentation does not still describe IAM detection as future work
- make sure “three alarms” has been updated to “four alarms” where relevant
- make sure “three detections” has been updated to “four detections” where relevant
- perform a final read-through for clarity, broken links, and unnecessary repetition

## Future Improvements

The core lab is complete. Future work would expand detection depth or improve analysis quality.

### Strong future detection candidates

CloudTrail-side candidates:

- failed AWS console login activity
- `CreateAccessKey` events
- `PutUserPolicy` events
- `AttachRolePolicy` events
- CloudTrail modification or disablement attempts
- suspicious administrative API activity

Host-side candidates:

- repeated failed password attempts for valid users
- successful SSH login detection
- anomalous sudo activity
- privilege escalation related log events

### Optional advanced improvements

- add a cleaner architecture diagram
- add an analyst triage playbook for each alert
- add severity labels for detections
- add MITRE ATT&CK mappings where appropriate
- improve correlation between host telemetry and AWS control-plane telemetry
- add automated response or remediation for selected alert types

## Current Priority

**Current phase:** Final documentation polish.

The AWS build and validation work is complete. The priority now is to make the GitHub documentation clean, consistent, and easy to defend in an interview.

## Open Notes

- The original Gmail-based SNS subscription path repeatedly deactivated after confirmation.
- This appeared to be an email endpoint or link-handling issue rather than an AWS alarm issue.
- End-to-end SNS delivery was successfully validated after switching to a different email endpoint.
- CloudTrail Event history was more reliable than CloudWatch log searching for quickly confirming the exact control-plane event name during detection engineering.
- Attempting to build failed AWS console login detection provided useful insight into sign-in event structure, but it was not the best implementation target for this phase compared with security group and IAM control-plane detections.
- CloudWatch metric graphs should generally be viewed with `Sum` as the statistic when validating event-count metrics.
- Screenshot organization should be checked before the lab is considered fully shipped.

## Final Goal

By the end of the lab, this project should clearly demonstrate:

- centralized monitoring across multiple AWS-relevant telemetry sources
- host-based detection engineering
- cloud control-plane detection engineering
- IAM control-plane detection engineering
- CloudWatch metric-filter-based detection
- CloudWatch-based alerting
- SNS-based notification delivery
- clear documentation and evidence for each stage

The final version should be strong enough to discuss in interviews as a real cloud security monitoring project rather than a basic AWS setup exercise.
