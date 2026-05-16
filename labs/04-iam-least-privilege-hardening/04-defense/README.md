# Defense

This section documents the least-privilege remediation and IAM hardening decisions made during the lab.

## Defensive Goal

The defensive goal was to reduce the permissions of the `lab4-junior-analyst` IAM user so the identity could only perform the actions required for its intended role.

The user should be able to:

- List the assigned Lab 4 S3 bucket
- Read objects inside the assigned Lab 4 S3 bucket

The user should not be able to:

- List all S3 buckets in the account
- Upload or delete S3 objects
- Modify bucket policies
- Enumerate IAM users
- Attach administrator permissions
- Create new access keys

## Initial Risk: Over-Permissioned S3 Access

The lab began with the `lab4-junior-analyst` user attached to the AWS managed `AmazonS3FullAccess` policy.

This was intentionally risky because the user only needed read access to one assigned bucket, but the policy allowed broad S3 access across the account.

From a cloud security perspective, this increased the potential blast radius of the identity. If the access key were misused, the identity could potentially interact with S3 resources outside its intended job function.

Evidence:

- `../screenshots/initial-overpermissive-policy-attached.png`
- `../screenshots/s3-access-allowed-before-remediation.png`
- `../screenshots/s3-object-upload-allowed-before-remediation.png`

## Remediation: Scoped Least-Privilege Policy

The broad managed policy was replaced with a custom least-privilege policy.

The new policy allowed only:

| Permission | Resource | Purpose |
|---|---|---|
| `s3:ListBucket` | `arn:aws:s3:::jw-lab4-iam-test-bucket-2026` | Allow the user to list the assigned bucket |
| `s3:GetObject` | `arn:aws:s3:::jw-lab4-iam-test-bucket-2026/*` | Allow the user to read objects in the assigned bucket |

Policy file:

- `../policies/least-privilege-s3-read-policy.json`

Evidence:

- `../screenshots/least-privilege-policy-created.png`
- `../screenshots/least-privilege-policy-attached.png`

## Least-Privilege Design Decisions

### Resource scoping

The policy was scoped to one specific S3 bucket instead of allowing account-wide S3 access.

This prevented the user from listing or interacting with unrelated S3 buckets.

### Action scoping

The policy allowed only the actions required for a read-only role:

- `s3:ListBucket`
- `s3:GetObject`

It did not allow write, delete, bucket policy modification, or IAM actions.

### Validation through denied actions

After remediation, I tested multiple actions that should not be allowed. These included S3 write/delete actions and IAM privilege escalation attempts.

Denied actions included:

- Listing all S3 buckets
- Uploading objects
- Deleting objects
- Modifying the S3 bucket policy
- Listing IAM users
- Attaching `AdministratorAccess`
- Creating new access keys

## Access Matrix

| Action Tested | After Remediation | Security Meaning |
|---|---|---|
| List assigned lab bucket | Allowed | Required read-only job access preserved |
| Download assigned object | Allowed | Required object read access preserved |
| List all S3 buckets | Denied | Prevents broad S3 enumeration |
| Upload object | Denied | Prevents unauthorized object writes |
| Delete object | Denied | Prevents destructive object actions |
| Modify bucket policy | Denied | Prevents weakening resource-based permissions |
| List IAM users | Denied | Prevents IAM identity enumeration |
| Attach `AdministratorAccess` | Denied | Prevents privilege escalation |
| Create new access key | Denied | Prevents credential persistence |

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

- `../screenshots/cloudtrail-session4-user-filtered-events.png`
- `../screenshots/cloudtrail-iam-list-users-access-denied.png`
- `../screenshots/cloudtrail-iam-attach-admin-policy-access-denied.png`
- `../screenshots/cloudtrail-iam-create-access-key-access-denied.png`
- `../screenshots/cloudtrail-s3-put-bucket-policy-access-denied.png`

## Defensive Outcome

The remediation successfully reduced the IAM user's permissions while preserving required access.

Before remediation, the user had broad S3 access through `AmazonS3FullAccess`.

After remediation, the user could still perform its intended read-only task, but unnecessary and higher-risk actions returned `AccessDenied`.

This demonstrated a complete least-privilege hardening workflow:

```text
Over-permissioned identity
→ risky access validated
→ scoped policy applied
→ allowed and denied actions tested
→ CloudTrail audit evidence collected
```

## Real-World Hardening Recommendations

In a production AWS environment, additional hardening would include:

- Use IAM groups or roles instead of attaching policies directly to individual users
- Require MFA for sensitive actions
- Monitor IAM policy changes with CloudWatch or EventBridge alerts
- Use IAM Access Analyzer to identify overly broad permissions
- Apply permission boundaries for high-risk users or automation accounts
- Rotate or remove unused access keys
- Review CloudTrail logs regularly for denied or unusual activity
- Avoid long-lived IAM user credentials when temporary role-based access is possible
