# Planning

## Purpose

This document tracks the working plan for the `cloud-security-monitoring` lab.

The goal of this lab is to build a centralized cloud security monitoring project in AWS that demonstrates visibility across both:

- host-level Linux authentication activity
- AWS control-plane activity

This file is meant to track progress, remaining work, and next priorities as the lab develops.

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

## Current Status

### Host-side monitoring path

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

### CloudTrail-side monitoring path

The CloudTrail-side monitoring path is also now built and validated end to end for one initial control-plane detection.

Completed:

- S3 bucket created for CloudTrail storage
- SNS topic created for alert delivery
- CloudTrail trail `cloud-security-monitoring-trail` created
- CloudTrail configured to deliver to S3
- CloudTrail configured to deliver to CloudWatch Logs
- CloudTrail telemetry validated in both S3 and CloudWatch
- CloudTrail log group `cloud-security-monitoring-cloudtrail` identified and validated
- controlled security group rule change generated against the EC2 instance’s attached security group
- CloudTrail event `RevokeSecurityGroupIngress` identified in Event history
- metric filter `revoke-security-group-ingress` created
- custom metric `CloudSecurityMonitoring / RevokeSecurityGroupIngressEvents` validated
- CloudWatch alarm `revoke-security-group-ingress-alarm` triggered successfully
- SNS email alert delivery validated

## Completed Documentation

The following sections are now drafted and updated to reflect both the host-side and CloudTrail-side work:

- `04-attack-simulation/README.md`
- `05-detection-engineering/README.md`
- `06-alerting-and-response/README.md`
- `07-reflections-and-improvements/README.md`

The root `README.md` has also been updated to reflect current lab status.

## Remaining Work

The biggest remaining gap is no longer whether the CloudTrail path works at all. It does.

The bigger remaining challenge is expanding the lab beyond one host-side detection and one CloudTrail-side detection so the project shows broader monitoring depth.

### Priority next steps

- add more CloudTrail-based detections beyond `RevokeSecurityGroupIngress`
- add more host-based detections beyond invalid-user SSH activity
- improve the consistency and polish of screenshots and diagrams
- strengthen cross-source explanation of how host and cloud telemetry complement each other
- continue refining documentation as the lab matures

### Strong next detection candidates

CloudTrail-side candidates:

- `AuthorizeSecurityGroupIngress`
- failed AWS console login activity
- IAM policy, role, or user changes
- suspicious administrative API activity
- CloudTrail modification or disablement attempts

Host-side candidates:

- repeated failed password attempts for valid users
- successful SSH login detection
- anomalous sudo activity
- privilege escalation related log events

## Current Priority

**Current phase:** Both the host-side monitoring path and one CloudTrail-side monitoring path are complete and validated.

**Next phase:** Expand the number and quality of detections so the lab demonstrates broader cloud security monitoring depth rather than just two validated signal paths.

## Open Notes

- The original Gmail-based SNS subscription path repeatedly deactivated after confirmation.
- This appeared to be an email endpoint or link-handling issue rather than an AWS alarm issue.
- End-to-end SNS delivery was successfully validated after switching to a different email endpoint.
- CloudTrail Event history was more reliable than CloudWatch log searching for quickly confirming the exact control-plane event name during detection engineering.
- Screenshot organization may still need cleanup depending on the final repo structure.

## Final Goal

By the end of the lab, this project should clearly demonstrate:

- centralized monitoring across multiple AWS-relevant telemetry sources
- host-based detection engineering
- cloud control-plane detection engineering
- CloudWatch-based alerting
- SNS-based notification delivery
- clear documentation and evidence for each stage

The final version should be strong enough to discuss in interviews as a real cloud security monitoring project rather than a basic AWS setup exercise.
