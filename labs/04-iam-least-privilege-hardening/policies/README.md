# IAM Policies

## Purpose

This folder contains the IAM policy evidence used in the AWS IAM least-privilege hardening lab.

The purpose of these files is to document the access-control change made during the lab:

```text
Over-permissioned S3 access
→ scoped least-privilege S3 read access
```

The policy files show the difference between the risky starting point and the remediated least-privilege design.

## Policy Files

| File | Purpose |
|---|---|
| [`initial-overpermissive-s3-policy.json`](initial-overpermissive-s3-policy.json) | Documents the initial risky policy state using the AWS managed `AmazonS3FullAccess` policy. |
| [`least-privilege-s3-read-policy.json`](least-privilege-s3-read-policy.json) | Contains the custom least-privilege policy that allows only read access to the assigned Lab 4 S3 bucket. |

## Initial Policy: Over-Permissioned Access

The lab began with the `lab4-junior-analyst` IAM user attached to the AWS managed policy:

```text
AmazonS3FullAccess
```

This was intentionally over-permissioned for the role.

The user only needed read access to one assigned S3 bucket, but `AmazonS3FullAccess` allowed broad S3 actions across the account.

This created unnecessary risk because a misused or compromised access key could potentially be used to:

- List S3 buckets across the account
- Upload objects
- Delete objects
- Interact with S3 resources outside the intended job function

Policy reference:

- [`initial-overpermissive-s3-policy.json`](initial-overpermissive-s3-policy.json)

Evidence:

- [`initial-overpermissive-policy-attached.png`](../screenshots/initial-overpermissive-policy-attached.png)
- [`s3-access-allowed-before-remediation.png`](../screenshots/s3-access-allowed-before-remediation.png)
- [`s3-object-upload-allowed-before-remediation.png`](../screenshots/s3-object-upload-allowed-before-remediation.png)

## Remediated Policy: Least-Privilege Read Access

The broad policy was replaced with a custom least-privilege policy.

The remediated policy allowed only:

| Permission | Resource | Purpose |
|---|---|---|
| `s3:ListBucket` | `arn:aws:s3:::jw-lab4-iam-test-bucket-2026` | Allow the user to list the assigned lab bucket |
| `s3:GetObject` | `arn:aws:s3:::jw-lab4-iam-test-bucket-2026/*` | Allow the user to read objects inside the assigned lab bucket |

Policy file:

- [`least-privilege-s3-read-policy.json`](least-privilege-s3-read-policy.json)

Evidence:

- [`least-privilege-policy-created.png`](../screenshots/least-privilege-policy-created.png)
- [`least-privilege-policy-attached.png`](../screenshots/least-privilege-policy-attached.png)

## Permissions Removed

The remediated policy did not grant permissions for:

- Listing all S3 buckets in the account
- Uploading objects
- Deleting objects
- Modifying bucket policies
- Listing IAM users
- Attaching new IAM policies
- Creating new access keys

These denied actions were validated through AWS CLI testing and CloudTrail review.

## Validation Results

| Action Tested | Expected Result | Actual Result |
|---|---:|---:|
| List assigned S3 bucket | Allowed | Allowed |
| Read object from assigned bucket | Allowed | Allowed |
| List all S3 buckets | Denied | Denied |
| Upload object | Denied | Denied |
| Delete object | Denied | Denied |
| Modify bucket policy | Denied | Denied |
| List IAM users | Denied | Denied |
| Attach `AdministratorAccess` | Denied | Denied |
| Create new access key | Denied | Denied |

## Security Outcome

After the least-privilege policy was applied, the IAM user could still perform the intended read-only task, but higher-risk S3 and IAM actions returned `AccessDenied`.

This reduced the user's blast radius and validated that the identity had only the permissions required for its assigned role.

## Key Takeaway

The policy change was not just a theoretical improvement.

It was validated through:

- AWS CLI testing
- Allowed-action confirmation
- Denied-action confirmation
- CloudTrail audit evidence
- Screenshot documentation

This demonstrated that least privilege was successfully applied and tested.
