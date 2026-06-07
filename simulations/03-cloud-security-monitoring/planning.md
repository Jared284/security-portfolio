# Planning

## Purpose

This document tracks the planning, scope, progress, and final status of the AWS Cloud Security Monitoring Lab.

The goal of this lab was to build a centralized cloud security monitoring project in AWS that demonstrates visibility across:

- Host-level Linux authentication activity
- AWS network-control-plane activity
- AWS identity-control-plane activity

This file documents the project scope, completed build work, validated detections, remaining documentation checks, and future improvement ideas.

---

## Lab Goal

The goal of this project was to prove that security-relevant telemetry from multiple sources can be centralized, monitored, converted into detection signals, and escalated into alerts.

The lab was designed to demonstrate the following workflow:

```text
Generate security-relevant activity
        ↓
Capture host or CloudTrail telemetry
        ↓
Centralize logs in CloudWatch Logs
        ↓
Create metric filters
        ↓
Generate custom CloudWatch metrics
        ↓
Trigger CloudWatch alarms
        ↓
Deliver SNS email notifications
```

---

## Lab Scope

This project focuses on AWS-native monitoring using CloudWatch, CloudTrail, EC2, and SNS.

### Primary Telemetry Sources

| Telemetry Source | Collection Method | Purpose |
|---|---|---|
| EC2 Linux authentication logs | CloudWatch Agent forwarding `/var/log/auth.log` | Host-side SSH monitoring |
| AWS CloudTrail events | CloudTrail delivery to S3 and CloudWatch Logs | AWS control-plane monitoring |

### Security Areas Covered

The lab covers three major areas of cloud security monitoring:

1. **Host-level monitoring**
   - Linux SSH authentication activity
   - Invalid-user login attempts
   - EC2 workload telemetry

2. **Network-control-plane monitoring**
   - AWS security group ingress rule additions
   - AWS security group ingress rule removals
   - CloudTrail-based network access change detection

3. **Identity-control-plane monitoring**
   - IAM managed policy attachment activity
   - Permission expansion visibility
   - CloudTrail-based IAM event detection

---

## Current Status

### Overall Status

The core AWS build and validation work is complete.

The lab now includes four validated detection and alerting paths:

1. Invalid-user SSH attempts from EC2 host logs
2. `RevokeSecurityGroupIngress` CloudTrail events
3. `AuthorizeSecurityGroupIngress` CloudTrail events
4. `AttachUserPolicy` CloudTrail events

Each path has been validated from event generation through:

- Log or CloudTrail event capture
- CloudWatch Logs ingestion
- Metric filter matching
- Custom metric creation
- CloudWatch alarm triggering
- SNS email notification delivery

---

## Host-Side Monitoring Path

The host-side monitoring path is built and validated end to end.

### Completed Work

- Deployed EC2 Ubuntu monitoring host
- Created and attached IAM role for CloudWatch Agent permissions
- Attached `CloudWatchAgentServerPolicy`
- Installed and configured the CloudWatch Agent
- Forwarded `/var/log/auth.log` into CloudWatch Logs
- Created CloudWatch log group `cloud-security-monitoring-auth`
- Simulated repeated fake SSH login attempts from local workstation
- Confirmed `Invalid user` entries in local host logs
- Confirmed `Invalid user` entries in CloudWatch Logs
- Created metric filter `invalid-user-ssh-attempts`
- Created custom metric `CloudSecurityMonitoring / InvalidUserSSHAttempts`
- Validated metric datapoints with `Sum` statistic
- Created CloudWatch alarm `invalid-user-ssh-attempts-alarm`
- Triggered alarm successfully
- Validated SNS email alert delivery

### Final Output

The host-side path detects repeated invalid-user SSH activity and sends an alert when the configured threshold is exceeded.

---

## CloudTrail-Side Monitoring Path

The CloudTrail-side monitoring path is built and validated across network-control-plane and identity-control-plane events.

### Completed Work

- Created S3 bucket for CloudTrail log storage
- Created SNS topic `cloud-security-monitoring-alerts`
- Created CloudTrail trail `cloud-security-monitoring-trail`
- Configured CloudTrail delivery to S3
- Configured CloudTrail delivery to CloudWatch Logs
- Created CloudWatch log group `cloud-security-monitoring-cloudtrail`
- Validated CloudTrail telemetry in S3
- Validated CloudTrail telemetry in CloudWatch Logs
- Generated controlled security group rule changes
- Identified `RevokeSecurityGroupIngress` in CloudTrail Event history
- Identified `AuthorizeSecurityGroupIngress` in CloudTrail Event history
- Generated controlled IAM managed policy attachment activity
- Identified `AttachUserPolicy` in CloudTrail Event history
- Created metric filter `revoke-security-group-ingress`
- Created metric filter `authorize-security-group-ingress`
- Created metric filter `attach-user-policy`
- Created custom metric `CloudSecurityMonitoring / RevokeSecurityGroupIngressEvents`
- Created custom metric `CloudSecurityMonitoring / AuthorizeSecurityGroupIngressEvents`
- Created custom metric `CloudSecurityMonitoring / AttachUserPolicyEvents`
- Created CloudWatch alarm `revoke-security-group-ingress-alarm`
- Created CloudWatch alarm `authorize-security-group-ingress-alarm`
- Created CloudWatch alarm `attach-user-policy-alarm`
- Triggered all CloudTrail-side alarms successfully
- Validated SNS email alert delivery for all CloudTrail-side detections

### Final Output

The CloudTrail-side path detects selected AWS administrative actions and sends alerts for security group changes and IAM managed policy attachment activity.

---

## Completed Detections

| # | Detection | Source | Metric Filter | Custom Metric | Alarm |
|---|---|---|---|---|---|
| 1 | Invalid SSH user attempts | EC2 `/var/log/auth.log` | `invalid-user-ssh-attempts` | `InvalidUserSSHAttempts` | `invalid-user-ssh-attempts-alarm` |
| 2 | Security group ingress rule removal | CloudTrail `RevokeSecurityGroupIngress` | `revoke-security-group-ingress` | `RevokeSecurityGroupIngressEvents` | `revoke-security-group-ingress-alarm` |
| 3 | Security group ingress rule addition | CloudTrail `AuthorizeSecurityGroupIngress` | `authorize-security-group-ingress` | `AuthorizeSecurityGroupIngressEvents` | `authorize-security-group-ingress-alarm` |
| 4 | IAM managed policy attachment | CloudTrail `AttachUserPolicy` | `attach-user-policy` | `AttachUserPolicyEvents` | `attach-user-policy-alarm` |

All custom metrics use the namespace:

```text
CloudSecurityMonitoring
```

---

## Completed Documentation

The following documentation sections have been drafted and polished:

- Root `README.md`
- `01-architecture/README.md`
- `02-setup-and-build/README.md`
- `03-log-sources-and-ingestion/README.md`
- `04-attack-simulation/README.md`
- `05-detection-engineering/README.md`
- `06-alerting-and-response/README.md`
- `07-reflections-and-improvements/README.md`

---

## Remaining Work

The main remaining work is final documentation and repository cleanup, not additional AWS implementation.

### Final Cleanup Checklist

- Confirm every screenshot path renders correctly in GitHub
- Confirm screenshot filenames match Markdown references
- Confirm there are no broken image links
- Remove duplicate or outdated screenshots if needed
- Confirm IAM detection is reflected consistently across all documentation
- Confirm all references say “four detections” where appropriate
- Confirm all references say “four alarms” where appropriate
- Confirm the main README matches the final lab structure
- Confirm `planning.md` reflects the final status of the project
- Perform final read-through for clarity and repetition
- Make sure no sensitive information is exposed in screenshots or text

---

## Future Improvements

The core lab is complete. Future work would expand detection coverage, improve analysis quality, or make the lab closer to a production-style detection engineering workflow.

### Strong Future Detection Candidates

CloudTrail-side candidates:

- Failed AWS console login activity
- `CreateAccessKey` events
- `PutUserPolicy` events
- `AttachRolePolicy` events
- CloudTrail modification or disablement attempts
- Root account activity
- Suspicious administrative API activity
- Unauthorized region activity

Host-side candidates:

- Repeated failed password attempts for valid users
- Successful SSH login detection
- Anomalous sudo activity
- Privilege escalation related log events
- New user creation on the host
- Changes to SSH configuration files

---

## Advanced Improvement Ideas

Optional advanced improvements include:

- Add a cleaner architecture diagram
- Add analyst triage playbooks for each alert
- Add severity labels for detections
- Add MITRE ATT&CK mappings where appropriate
- Improve correlation between host telemetry and AWS control-plane telemetry
- Add automated response or remediation for selected alert types
- Integrate logs with a SIEM such as Splunk, Elastic, or Microsoft Sentinel
- Convert the lab build into Infrastructure as Code using Terraform or CloudFormation

---

## Notes and Lessons Learned

Important lessons from the project:

- The original Gmail-based SNS subscription path repeatedly deactivated after confirmation.
- This appeared to be an email endpoint or link-handling issue rather than an AWS alarm issue.
- End-to-end SNS delivery was successfully validated after switching to a different email endpoint.
- CloudTrail Event history was more reliable than CloudWatch log searching for quickly confirming exact control-plane event names.
- Failed AWS console login detection was considered, but it was not the cleanest implementation target for this phase.
- Security group changes and IAM policy attachment produced clearer, more defensible detection paths.
- CloudWatch metric graphs should generally be viewed with the `Sum` statistic when validating event-count metrics.
- Screenshot organization and image rendering should be checked before the lab is considered fully shipped.

---

## Final Goal

The final version of this project should clearly demonstrate:

- Centralized monitoring across multiple AWS-relevant telemetry sources
- Host-based detection engineering
- Cloud control-plane detection engineering
- IAM control-plane detection engineering
- CloudWatch metric-filter-based detection
- CloudWatch alarm configuration
- SNS-based notification delivery
- Troubleshooting of alert delivery paths
- Clear documentation and screenshot evidence for each stage

The final version should be strong enough to discuss in interviews as a real cloud security monitoring project rather than a basic AWS setup exercise.

---

## Final Status

The AWS build and validation work is complete.

The project is now in final documentation polish mode.

The remaining priority is to make the GitHub documentation clean, consistent, and easy to defend in an interview.
