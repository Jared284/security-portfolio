# IAM Policies

This folder contains the IAM policy evidence used in this lab.

The purpose of these files is to show the access-control change made during the lab: starting with an intentionally over-permissioned S3 policy and then replacing it with a scoped least-privilege policy.

## Policy Files

| File | Purpose |
|---|---|
| `initial-overpermissive-s3-policy.json` | Documents the initial risky policy state using the AWS managed `AmazonS3FullAccess` policy. |
| `least-privilege-s3-read-policy.json` | Contains the custom least-privilege policy that allows only read access to the assigned Lab 4 S3 bucket. |

## Initial Policy: Over-Permissioned Access

The lab began with the `lab4-junior-analyst` IAM user attached to the AWS managed `AmazonS3FullAccess` policy.

This was intentionally over-permissioned for the role. The user only needed read access to one assigned lab bucket, but `AmazonS3FullAccess` allowed broad S3 actions across the account.

This created unnecessary risk because a misused or compromised access key could potentially be used to list buckets, upload objects, delete objects, or interact with S3 resources outside the intended job function.

## Remediated Policy: Least-Privilege Read Access

The broad policy was replaced with a custom policy that only allowed:

- `s3:ListBucket` on `jw-lab4-iam-test-bucket-2026`
- `s3:GetObject` on objects inside `jw-lab4-iam-test-bucket-2026`

The policy did not grant permissions for:

- Listing all S3 buckets in the account
- Uploading objects
- Deleting objects
- Modifying bucket policies
- Listing IAM users
- Attaching new IAM policies
- Creating new access keys

## Security Outcome

After the least-privilege policy was applied, the IAM user could still perform the intended read-only task, but higher-risk S3 and IAM actions returned `AccessDenied`.

This reduced the user's blast radius and validated that the identity had only the permissions required for its assigned role.
