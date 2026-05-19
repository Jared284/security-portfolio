# Screenshots

## Purpose

This folder contains screenshot evidence collected during the AWS IAM least-privilege hardening lab.

The screenshots document the full validation workflow:

```text
IAM user created
→ over-permissioned policy attached
→ broad S3 access confirmed
→ least-privilege policy applied
→ required access preserved
→ risky actions denied
→ CloudTrail audit evidence collected
```

These screenshots support the technical claims made throughout the lab documentation.

## Evidence Categories

| Category | Purpose |
|---|---|
| Setup evidence | Shows the IAM user, S3 bucket, and test object used in the lab |
| Baseline access evidence | Shows the initial over-permissioned state before remediation |
| Remediation evidence | Shows the least-privilege policy being created and attached |
| Allowed-action evidence | Shows required read access still working after remediation |
| Denied-action evidence | Shows risky S3 and IAM actions being blocked |
| CloudTrail evidence | Shows denied activity recorded in AWS audit logs |

## Setup Evidence

| Screenshot | What It Shows |
|---|---|
| [`iam-user-lab4-junior-analyst-created.png`](iam-user-lab4-junior-analyst-created.png) | The dedicated IAM test user created for the lab |
| [`s3-lab-bucket-created.png`](s3-lab-bucket-created.png) | The assigned S3 bucket used for access testing |
| [`s3-test-object-uploaded.png`](s3-test-object-uploaded.png) | The test object uploaded to the lab bucket |
| [`aws-cli-get-caller-identity.png`](aws-cli-get-caller-identity.png) | AWS CLI identity validation for the `lab4-junior-analyst` user |

## Baseline Over-Permissioned Access Evidence

| Screenshot | What It Shows |
|---|---|
| [`initial-overpermissive-policy-attached.png`](initial-overpermissive-policy-attached.png) | The initial `AmazonS3FullAccess` policy attached to the IAM user |
| [`s3-access-allowed-before-remediation.png`](s3-access-allowed-before-remediation.png) | The user could list S3 buckets before remediation |
| [`s3-object-upload-allowed-before-remediation.png`](s3-object-upload-allowed-before-remediation.png) | The user could upload an object before remediation |

## Least-Privilege Remediation Evidence

| Screenshot | What It Shows |
|---|---|
| [`least-privilege-policy-created.png`](least-privilege-policy-created.png) | The custom least-privilege S3 read policy created |
| [`least-privilege-policy-attached.png`](least-privilege-policy-attached.png) | The least-privilege policy attached to the IAM user |

## Required Access Preserved

| Screenshot | What It Shows |
|---|---|
| [`s3-read-access-allowed-after-remediation.png`](s3-read-access-allowed-after-remediation.png) | The user could still read from the assigned S3 bucket after remediation |
| [`session3-s3-read-access-allowed.png`](session3-s3-read-access-allowed.png) | Additional validation that assigned read access still worked |

## Denied S3 Action Evidence

| Screenshot | What It Shows |
|---|---|
| [`s3-list-all-buckets-denied-after-remediation.png`](s3-list-all-buckets-denied-after-remediation.png) | The user could no longer list all S3 buckets |
| [`s3-upload-denied-after-remediation.png`](s3-upload-denied-after-remediation.png) | The user could no longer upload objects after remediation |
| [`session3-s3-list-all-buckets-denied.png`](session3-s3-list-all-buckets-denied.png) | Additional validation that broad S3 listing was denied |
| [`session3-s3-upload-denied.png`](session3-s3-upload-denied.png) | Additional validation that S3 upload was denied |
| [`session3-s3-delete-denied.png`](session3-s3-delete-denied.png) | The user could not delete objects |
| [`session3-s3-put-bucket-policy-denied.png`](session3-s3-put-bucket-policy-denied.png) | The user could not modify the S3 bucket policy |

## Denied IAM Action Evidence

| Screenshot | What It Shows |
|---|---|
| [`iam-list-users-denied-after-remediation.png`](iam-list-users-denied-after-remediation.png) | The user could not list IAM users |
| [`session3-iam-attach-admin-policy-denied.png`](session3-iam-attach-admin-policy-denied.png) | The user could not attach `AdministratorAccess` to itself |
| [`session3-iam-create-access-key-denied.png`](session3-iam-create-access-key-denied.png) | The user could not create a new access key |

## CloudTrail Audit Evidence

| Screenshot | What It Shows |
|---|---|
| [`cloudtrail-session4-user-filtered-events.png`](cloudtrail-session4-user-filtered-events.png) | CloudTrail events filtered for the `lab4-junior-analyst` user |
| [`cloudtrail-iam-list-users-access-denied.png`](cloudtrail-iam-list-users-access-denied.png) | CloudTrail record of denied `ListUsers` activity |
| [`cloudtrail-iam-attach-admin-policy-access-denied.png`](cloudtrail-iam-attach-admin-policy-access-denied.png) | CloudTrail record of denied `AttachUserPolicy` activity |
| [`cloudtrail-iam-create-access-key-access-denied.png`](cloudtrail-iam-create-access-key-access-denied.png) | CloudTrail record of denied `CreateAccessKey` activity |
| [`cloudtrail-s3-put-bucket-policy-access-denied.png`](cloudtrail-s3-put-bucket-policy-access-denied.png) | CloudTrail record of denied `PutBucketPolicy` activity |

## Evidence Summary

| Phase | Evidence |
|---|---|
| Setup | IAM user, S3 bucket, test object, AWS CLI identity |
| Baseline | Broad S3 access confirmed before remediation |
| Remediation | Least-privilege policy created and attached |
| Validation | Required access allowed, risky access denied |
| Audit | Denied actions recorded in CloudTrail |

## Final Takeaway

The screenshot evidence proves that the lab was not just theoretical.

The screenshots show the complete IAM hardening lifecycle:

```text
Create identity
→ test excessive access
→ apply least privilege
→ validate allowed and denied actions
→ confirm audit evidence in CloudTrail
```

This evidence supports the final conclusion that the `lab4-junior-analyst` user was reduced to the permissions required for its assigned role.
