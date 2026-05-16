# Architecture

## Purpose

This section documents the high-level architecture for the AWS IAM least-privilege hardening lab.

The lab is built around a test IAM identity that initially has broader permissions than required. Access is tested through the AWS CLI, logged through CloudTrail, and then remediated with a scoped least-privilege policy.

## Lab Architecture Summary

The core architecture follows a simple security validation workflow:

```text
Over-permissioned IAM identity
→ AWS CLI access testing
→ S3 and IAM action validation
→ least-privilege remediation
→ denied-action testing
→ CloudTrail audit evidence
```

The goal is to show how an AWS identity's permissions affect what actions it can perform, how excessive access can increase risk, and how least-privilege policy design reduces that risk.

## Main Components

| Component | Purpose |
|---|---|
| `lab4-junior-analyst` IAM user | Test identity used to simulate a restricted cloud user |
| `AmazonS3FullAccess` | Initial intentionally over-permissioned policy |
| Custom least-privilege S3 policy | Remediated policy allowing only required read access |
| S3 bucket | Assigned lab resource used for access testing |
| AWS CLI | Used to test allowed and denied actions from the user's perspective |
| CloudTrail | Used to validate that AWS recorded the user's allowed and denied API activity |
| Screenshots | Evidence of configuration, CLI results, and CloudTrail audit logs |

## High-Level Flow

```text
+--------------------------------+
| IAM User                       |
| lab4-junior-analyst            |
+----------------+---------------+
                 |
                 v
+--------------------------------+
| Initial Policy                 |
| AmazonS3FullAccess             |
| Intentionally over-permissioned|
+----------------+---------------+
                 |
                 v
+--------------------------------+
| Baseline Access Testing        |
| AWS CLI S3 commands            |
| - List buckets                 |
| - Upload object                |
+----------------+---------------+
                 |
                 v
+--------------------------------+
| Risk Identified                |
| User has broader access        |
| than required for role         |
+----------------+---------------+
                 |
                 v
+--------------------------------+
| Least-Privilege Remediation    |
| Custom scoped S3 read policy   |
| - s3:ListBucket                |
| - s3:GetObject                 |
+----------------+---------------+
                 |
                 v
+--------------------------------+
| Post-Remediation Testing       |
| Validate allowed/denied access |
| through AWS CLI                |
+----------------+---------------+
                 |
                 v
+--------------------------------+
| CloudTrail Audit Evidence      |
| AccessDenied events recorded   |
| with user, event, time, IP     |
+--------------------------------+
```

## Before Remediation

Before remediation, the test IAM user had the AWS managed `AmazonS3FullAccess` policy attached.

This allowed the user to perform broad S3 actions beyond the intended role.

The user was able to:

- Authenticate through the AWS CLI
- List S3 buckets in the account
- Upload an object to the lab S3 bucket

This represented an over-permissioned identity because the user only needed read access to one assigned bucket.

## After Remediation

After remediation, the broad managed policy was replaced with a custom least-privilege policy.

The remediated policy allowed only:

- `s3:ListBucket` on the assigned lab bucket
- `s3:GetObject` on objects inside the assigned lab bucket

The remediated policy did not allow:

- Listing all S3 buckets
- Uploading objects
- Deleting objects
- Modifying bucket policies
- Listing IAM users
- Attaching administrator permissions
- Creating new access keys

## Access-Control Flow

```text
User identity
   |
   v
IAM policy evaluation
   |
   v
Requested AWS action
   |
   v
Target AWS resource
   |
   v
AWS decision: Allow or Deny
   |
   v
CloudTrail audit event
```

This lab demonstrates that AWS access is determined by the combination of:

```text
Identity + Policy + Action + Resource = Access Decision
```

## Security Boundary

The intended security boundary was:

```text
lab4-junior-analyst
→ read-only access
→ one assigned S3 bucket
```

The user should not have account-wide S3 visibility or IAM-level permissions.

## CloudTrail Role in the Architecture

CloudTrail provided the audit layer for the lab.

After denied S3 and IAM actions were tested, CloudTrail was used to confirm that AWS recorded the activity with fields such as:

- Event name
- User identity
- Event source
- Source IP address
- Event time
- Error code

This connected the access-control tests to real AWS audit evidence.

## Final Architecture Outcome

The final architecture demonstrated a complete IAM hardening workflow:

```text
Over-permissioned identity
→ risky access validated
→ scoped least-privilege policy applied
→ risky actions denied
→ CloudTrail audit evidence collected
```

The result was a restricted IAM user that could still perform its intended read-only task while unnecessary S3 and IAM permissions were removed.


## Architecture Evidence

The following screenshots support the architecture and access-control flow documented above:

| Evidence | Screenshot |
|---|---|
| IAM user used for testing | [`iam-user-lab4-junior-analyst-created.png`](../screenshots/iam-user-lab4-junior-analyst-created.png) |
| Initial over-permissioned policy | [`initial-overpermissive-policy-attached.png`](../screenshots/initial-overpermissive-policy-attached.png) |
| AWS CLI identity validation | [`aws-cli-get-caller-identity.png`](../screenshots/aws-cli-get-caller-identity.png) |
| Least-privilege policy attached | [`least-privilege-policy-attached.png`](../screenshots/least-privilege-policy-attached.png) |
| CloudTrail audit evidence | [`cloudtrail-session4-user-filtered-events.png`](../screenshots/cloudtrail-session4-user-filtered-events.png) |
