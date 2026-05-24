# 01 - Architecture

## Purpose

This section documents the high-level architecture for the AWS IAM Least Privilege and Access Control Hardening Lab.

The lab is built around a dedicated IAM test identity that initially has broader permissions than required. That access is tested through the AWS CLI, remediated with a scoped least-privilege policy, retested after remediation, and validated with CloudTrail audit evidence.

The goal is to show how IAM permissions affect real AWS actions and how least-privilege design reduces the blast radius of an identity.

---

## Architecture Summary

The lab follows a complete IAM hardening workflow:

```text
Over-permissioned IAM identity
        ↓
AWS CLI access testing
        ↓
Risky access validated
        ↓
Broad policy removed
        ↓
Least-privilege policy applied
        ↓
Allowed and denied actions retested
        ↓
CloudTrail audit evidence reviewed
```

This architecture shows the relationship between identity, policy, action, resource, access decision, and audit logging.

---

## Main Components

| Component | Purpose |
|---|---|
| `lab4-junior-analyst` IAM user | Test identity used to simulate a restricted cloud user |
| `AmazonS3FullAccess` | Initial intentionally over-permissioned AWS managed policy |
| Custom least-privilege S3 policy | Remediated policy allowing only required read access |
| S3 lab bucket | Assigned resource used for access testing |
| AWS CLI | Used to test allowed and denied actions from the user’s perspective |
| CloudTrail | Used to confirm AWS recorded the user’s allowed and denied API activity |
| Screenshots | Evidence of configuration, CLI results, and CloudTrail audit logs |

---

## High-Level Architecture

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
| User has broader S3 access     |
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
| Validate allowed access        |
| Validate denied actions        |
+----------------+---------------+
                 |
                 v
+--------------------------------+
| CloudTrail Audit Evidence      |
| AccessDenied events recorded   |
| with user, event, time, IP     |
+--------------------------------+
```

---

## Before Remediation

Before remediation, the test IAM user had the AWS managed `AmazonS3FullAccess` policy attached.

This gave the user broader S3 access than the intended role required.

The user was able to:

- Authenticate through the AWS CLI
- List S3 buckets in the account
- Upload an object to the lab S3 bucket

This represented an over-permissioned identity because the intended role only required read access to one assigned S3 bucket.

---

## After Remediation

After remediation, the broad managed policy was removed and replaced with a custom least-privilege policy.

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

This reduced the user’s effective permissions while preserving the access needed for the intended role.

---

## Access-Control Flow

```text
User identity
        ↓
IAM policy evaluation
        ↓
Requested AWS action
        ↓
Target AWS resource
        ↓
AWS access decision
        ↓
CloudTrail audit event
```

This lab demonstrates that AWS access decisions are based on the relationship between:

```text
Identity + Policy + Action + Resource = Access Decision
```

---

## Intended Security Boundary

The intended security boundary was:

```text
lab4-junior-analyst
        ↓
Read-only access
        ↓
One assigned S3 bucket
```

The user should not have:

- Account-wide S3 visibility
- S3 write access
- S3 destructive access
- S3 bucket policy modification access
- IAM enumeration permissions
- IAM privilege escalation permissions
- Access key creation permissions

---

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

CloudTrail mattered because the lab was not only about blocking risky actions. It was also about proving that denied activity could be reviewed later during investigation.

---

## Architecture Evidence

The following screenshots support the architecture and access-control flow documented above:

| Evidence | Screenshot |
|---|---|
| IAM user used for testing | [`iam-user-lab4-junior-analyst-created.png`](../screenshots/iam-user-lab4-junior-analyst-created.png) |
| Initial over-permissioned policy | [`initial-overpermissive-policy-attached.png`](../screenshots/initial-overpermissive-policy-attached.png) |
| AWS CLI identity validation | [`aws-cli-get-caller-identity.png`](../screenshots/aws-cli-get-caller-identity.png) |
| Least-privilege policy attached | [`least-privilege-policy-attached.png`](../screenshots/least-privilege-policy-attached.png) |
| CloudTrail audit evidence | [`cloudtrail-session4-user-filtered-events.png`](../screenshots/cloudtrail-session4-user-filtered-events.png) |

---

## Final Architecture Outcome

The final architecture demonstrated a complete IAM hardening workflow:

```text
Over-permissioned identity
        ↓
Risky access validated
        ↓
Scoped least-privilege policy applied
        ↓
Required access preserved
        ↓
Risky actions denied
        ↓
CloudTrail audit evidence collected
```

The result was a restricted IAM user that could still perform its intended read-only task while unnecessary S3 and IAM permissions were removed.
