# Defense

## Purpose

This section documents the defensive decisions made during the AWS IAM least-privilege hardening lab.

The goal was to reduce the permissions of the `lab4-junior-analyst` IAM user so the identity could only perform the actions required for its intended role.

This lab focuses on practical IAM defense:

```text
Identify excessive access
→ remove broad permissions
→ apply scoped least privilege
→ validate required access
→ confirm risky actions are denied
→ verify activity in CloudTrail
```

## Defensive Goal

The `lab4-junior-analyst` user should be able to perform only one basic job function:

```text
Read from one assigned S3 bucket
```

The user should be able to:

- List the assigned Lab 4 S3 bucket
- Read objects inside the assigned Lab 4 S3 bucket

The user should not be able to:

- List all S3 buckets in the account
- Upload S3 objects
- Delete S3 objects
- Modify S3 bucket policies
- Enumerate IAM users
- Attach administrator permissions
- Create new access keys

## Initial Risk: Over-Permissioned S3 Access

The lab began with the `lab4-junior-analyst` user attached to the AWS managed policy:

```text
AmazonS3FullAccess
```

This was intentionally over-permissioned.

The user only needed read access to one assigned S3 bucket, but `AmazonS3FullAccess` allowed broad S3 actions across the account.

From a cloud security perspective, this increased the identity's blast radius. If the access key were misused, the identity could potentially interact with S3 resources outside its intended job function.

Evidence:

- [`initial-overpermissive-policy-attached.png`](../screenshots/initial-overpermissive-policy-attached.png)
- [`s3-access-allowed-before-remediation.png`](../screenshots/s3-access-allowed-before-remediation.png)
- [`s3-object-upload-allowed-before-remediation.png`](../screenshots/s3-object-upload-allowed-before-remediation.png)

## Why the Initial Policy Was Risky

The initial policy was risky because it violated least privilege.

A junior analyst-style identity did not need broad S3 permissions. It only needed limited read access to one assigned bucket.

The broad policy created several risks:

| Risk | Why It Matters |
|---|---|
| Account-wide S3 visibility | The user could list buckets outside the assigned scope |
| Unnecessary write access | The user could upload objects even though the role was intended to be read-only |
| Larger blast radius | If credentials were misused, more AWS resources could be affected |
| Weak role separation | The user's permissions exceeded its intended job function |
| Harder investigation scope | More permissions create more possible paths of misuse |

## Remediation: Scoped Least-Privilege Policy

The broad managed policy was replaced with a custom least-privilege policy.

The new policy allowed only:

| Permission | Resource | Purpose |
|---|---|---|
| `s3:ListBucket` | `arn:aws:s3:::jw-lab4-iam-test-bucket-2026` | Allow the user to list the assigned bucket |
| `s3:GetObject` | `arn:aws:s3:::jw-lab4-iam-test-bucket-2026/*` | Allow the user to read objects in the assigned bucket |

Policy file:

- [`least-privilege-s3-read-policy.json`](../policies/least-privilege-s3-read-policy.json)

Evidence:

- [`least-privilege-policy-created.png`](../screenshots/least-privilege-policy-created.png)
- [`least-privilege-policy-attached.png`](../screenshots/least-privilege-policy-attached.png)

## Least-Privilege Design Decisions

### 1. Scope access to one resource

The policy was scoped to one specific S3 bucket:

```text
jw-lab4-iam-test-bucket-2026
```

This prevented the user from listing or interacting with unrelated S3 buckets.

### 2. Allow only required actions

The user only needed read access.

The remediated policy allowed:

```text
s3:ListBucket
s3:GetObject
```

The remediated policy did not allow:

```text
s3:PutObject
s3:DeleteObject
s3:ListAllMyBuckets
s3:PutBucketPolicy
iam:ListUsers
iam:AttachUserPolicy
iam:CreateAccessKey
```

### 3. Preserve required access

A strong least-privilege policy should not break the user's required job function.

After remediation, the user could still list the assigned bucket and download the assigned object.

Evidence:

- [`s3-read-access-allowed-after-remediation.png`](../screenshots/s3-read-access-allowed-after-remediation.png)
- [`session3-s3-read-access-allowed.png`](../screenshots/session3-s3-read-access-allowed.png)

### 4. Validate denied access

The remediation was not assumed to work. It was validated by testing actions the user should not be able to perform.

Denied actions included:

- Listing all S3 buckets
- Uploading objects
- Deleting objects
- Modifying the bucket policy
- Listing IAM users
- Attaching `AdministratorAccess`
- Creating new access keys

## Access Matrix

| Action Tested | Intended for Role? | Result After Remediation | Security Meaning |
|---|---:|---:|---|
| List assigned lab bucket | Yes | Allowed | Required read-only job access preserved |
| Download assigned object | Yes | Allowed | Required object read access preserved |
| List all S3 buckets | No | Denied | Prevents broad S3 enumeration |
| Upload object | No | Denied | Prevents unauthorized object writes |
| Delete object | No | Denied | Prevents destructive object actions |
| Modify bucket policy | No | Denied | Prevents weakening resource-based permissions |
| List IAM users | No | Denied | Prevents IAM identity enumeration |
| Attach `AdministratorAccess` | No | Denied | Prevents privilege escalation |
| Create new access key | No | Denied | Prevents credential persistence |

## S3 Defensive Validation

The S3 validation confirmed that unnecessary storage permissions were removed.

### Broad bucket listing denied

Evidence:

- [`s3-list-all-buckets-denied-after-remediation.png`](../screenshots/s3-list-all-buckets-denied-after-remediation.png)
- [`session3-s3-list-all-buckets-denied.png`](../screenshots/session3-s3-list-all-buckets-denied.png)

Security meaning:

The user could no longer enumerate all buckets in the account.

### Upload denied

Evidence:

- [`s3-upload-denied-after-remediation.png`](../screenshots/s3-upload-denied-after-remediation.png)
- [`session3-s3-upload-denied.png`](../screenshots/session3-s3-upload-denied.png)

Security meaning:

The user could no longer write new objects to the bucket.

### Delete denied

Evidence:

- [`session3-s3-delete-denied.png`](../screenshots/session3-s3-delete-denied.png)

Security meaning:

The user could not perform destructive object actions.

### Bucket policy modification denied

Evidence:

- [`session3-s3-put-bucket-policy-denied.png`](../screenshots/session3-s3-put-bucket-policy-denied.png)
- [`cloudtrail-s3-put-bucket-policy-access-denied.png`](../screenshots/cloudtrail-s3-put-bucket-policy-access-denied.png)

Security meaning:

The user could not weaken the bucket's resource-based permissions.

## IAM Defensive Validation

The IAM validation confirmed that the user could not perform actions related to reconnaissance, privilege escalation, or credential persistence.

### IAM enumeration denied

Evidence:

- [`iam-list-users-denied-after-remediation.png`](../screenshots/iam-list-users-denied-after-remediation.png)
- [`cloudtrail-iam-list-users-access-denied.png`](../screenshots/cloudtrail-iam-list-users-access-denied.png)

Security meaning:

The user could not list IAM users in the account.

### Administrator policy attachment denied

Evidence:

- [`session3-iam-attach-admin-policy-denied.png`](../screenshots/session3-iam-attach-admin-policy-denied.png)
- [`cloudtrail-iam-attach-admin-policy-access-denied.png`](../screenshots/cloudtrail-iam-attach-admin-policy-access-denied.png)

Security meaning:

The user could not attach `AdministratorAccess` to itself. This prevented a direct privilege escalation path.

### Access key creation denied

Evidence:

- [`session3-iam-create-access-key-denied.png`](../screenshots/session3-iam-create-access-key-denied.png)
- [`cloudtrail-iam-create-access-key-access-denied.png`](../screenshots/cloudtrail-iam-create-access-key-access-denied.png)

Security meaning:

The user could not create additional long-lived credentials. This reduced credential persistence risk.

## CloudTrail Validation

CloudTrail was used to verify that denied S3 and IAM actions were recorded as audit events.

CloudTrail evidence showed:

- Event name
- User identity
- Event source
- Source IP address
- Event time
- `AccessDenied` error code

Evidence:

- [`cloudtrail-session4-user-filtered-events.png`](../screenshots/cloudtrail-session4-user-filtered-events.png)
- [`cloudtrail-iam-list-users-access-denied.png`](../screenshots/cloudtrail-iam-list-users-access-denied.png)
- [`cloudtrail-iam-attach-admin-policy-access-denied.png`](../screenshots/cloudtrail-iam-attach-admin-policy-access-denied.png)
- [`cloudtrail-iam-create-access-key-access-denied.png`](../screenshots/cloudtrail-iam-create-access-key-access-denied.png)
- [`cloudtrail-s3-put-bucket-policy-access-denied.png`](../screenshots/cloudtrail-s3-put-bucket-policy-access-denied.png)

CloudTrail mattered because it proved the activity was not only blocked, but also logged for investigation and accountability.

## Defensive Outcome

The remediation successfully reduced the IAM user's permissions while preserving required access.

Before remediation:

```text
lab4-junior-analyst
→ AmazonS3FullAccess
→ broad S3 access across the account
```

After remediation:

```text
lab4-junior-analyst
→ custom least-privilege policy
→ read-only access to one assigned bucket
```

This demonstrated a complete least-privilege hardening workflow:

```text
Over-permissioned identity
→ risky access validated
→ scoped policy applied
→ required access preserved
→ risky actions denied
→ CloudTrail audit evidence collected
```

## Real-World Hardening Recommendations

In a production AWS environment, additional hardening would include:

- Use IAM groups, roles, or identity federation instead of attaching policies directly to individual users
- Prefer temporary role-based credentials over long-lived IAM user access keys
- Require MFA for sensitive actions
- Monitor IAM policy changes with EventBridge, CloudWatch, or a SIEM
- Alert on high-risk events such as `AttachUserPolicy`, `CreateAccessKey`, and `PutBucketPolicy`
- Use IAM Access Analyzer to identify overly broad permissions
- Apply permission boundaries for high-risk users or automation accounts
- Rotate or remove unused access keys
- Review CloudTrail logs regularly for denied or unusual activity
- Avoid assigning AWS managed full-access policies unless there is a clear operational requirement

## Final Takeaway

Least privilege is not only about writing a smaller policy.

It must be validated.

This lab showed that the IAM user could still perform the intended read-only S3 task, while unnecessary S3 actions, privilege escalation attempts, and credential persistence attempts were denied and logged in CloudTrail.
