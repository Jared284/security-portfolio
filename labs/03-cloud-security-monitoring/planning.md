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

The host-side monitoring path is now built and validated end to end.

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

### CloudTrail ingestion path

The CloudTrail ingestion path is built, but the detection side is not yet complete.

Completed:

- S3 bucket created for CloudTrail storage
- SNS topic created for alert delivery
- CloudTrail trail `cloud-security-monitoring-trail` created
- CloudTrail configured to deliver to S3
- CloudTrail configured to deliver to CloudWatch Logs
- CloudTrail telemetry validated in both S3 and CloudWatch

Not yet completed:

- CloudTrail-side detection engineering
- CloudTrail-based alarming
- CloudTrail-specific attack/admin activity simulation and validation

## Completed Documentation

The following sections are now drafted:

- `04-attack-simulation/README.md`
- `05-detection-engineering/README.md`
- `06-alerting-and-response/README.md`
- `07-reflections-and-improvements/README.md`

The root `README.md` has also been updated to reflect current lab status.

## Remaining Work

The biggest remaining gap is the cloud-side detection path.

### Priority next steps

- build at least one meaningful CloudTrail-based detection
- validate that CloudTrail-side detection with real test activity
- create alarming for the CloudTrail-side signal
- document the CloudTrail detection path in the same style as the host-side path

### Possible CloudTrail detections

Strong candidates for the next detection include:

- security group modifications
- failed AWS console logins
- IAM policy, role, or user changes
- suspicious administrative API activity
- trail modification or disablement attempts

## Current Priority

**Current phase:** Host-based monitoring path complete and validated.

**Next phase:** Build the CloudTrail-side detection path so the lab demonstrates both host-level and cloud-level security monitoring.

## Open Notes

- The original Gmail-based SNS subscription path repeatedly deactivated after confirmation.
- This appeared to be an email endpoint or link-handling issue rather than an AWS alarm issue.
- End-to-end SNS delivery was successfully validated after switching to a different email endpoint.
- Screenshot organization may still need cleanup depending on final repo structure.

## Final Goal

By the end of the lab, this project should clearly demonstrate:

- centralized monitoring across multiple AWS-relevant telemetry sources
- host-based detection engineering
- cloud control-plane detection engineering
- CloudWatch-based alerting
- SNS-based notification delivery
- clear documentation and evidence for each stage

The final version should be strong enough to discuss in interviews as a real cloud security monitoring project rather than a basic AWS setup exercise.
