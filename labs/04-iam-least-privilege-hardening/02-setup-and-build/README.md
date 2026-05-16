# Setup and Build

## Purpose

This section documents the AWS environment setup for the IAM least-privilege hardening lab.

The setup phase created the test IAM user, assigned S3 bucket, baseline over-permissioned access, AWS CLI profile, least-privilege replacement policy, and CloudTrail audit foundation needed to validate the remediation.

The goal was to build a controlled AWS environment where permissions could be tested before and after least-privilege hardening.

## Build Summary

```text
IAM User: lab4-junior-analyst
Assigned S3 Bucket: jw-lab4-iam-test-bucket-2026
Initial Policy: AmazonS3FullAccess
Remediated Policy: Custom least-privilege S3 read policy
Audit Layer: AWS CloudTrail
Testing Method: AWS CLI
```

## Environment Design

The lab was designed around one restricted test identity and one assigned S3 resource.

```text
lab4-junior-analyst
        |
        v
Initial broad S3 policy
        |
        v
S3 access testing through AWS CLI
        |
        v
Least-privilege policy remediation
        |
        v
Allowed and denied action validation
        |
        v
CloudTrail audit review
```

This setup made it possible to prove the difference between broad access and scoped access through actual AWS CLI testing.

## Step 1: Create the S3 Lab Bucket

A dedicated S3 bucket was created for the lab:

```text
jw-lab4-iam-test-bucket-2026
```

This bucket acted as the assigned resource for the `lab4-junior-analyst` user.

Using a dedicated bucket kept the lab controlled and prevented testing from affecting unrelated AWS resources.

Evidence:

- [`s3-lab-bucket-created.png`](../screenshots/s3-lab-bucket-created.png)

## Step 2: Upload a Test Object

A harmless test file was created and uploaded to the lab bucket.

Example file:

```text
lab4-test-file.txt
```

Example file content:

```text
This is a harmless test file for Lab 4 IAM least-privilege access testing.
```

This object was used to test whether the IAM user could read from the assigned bucket after least-privilege remediation.

Evidence:

- [`s3-test-object-uploaded.png`](../screenshots/s3-test-object-uploaded.png)

## Step 3: Create the Test IAM User

A dedicated IAM user was created:

```text
lab4-junior-analyst
```

This user represented a junior analyst identity that should only need read access to one assigned S3 bucket.

Using a separate test identity was important because permissions needed to be tested from the restricted user's perspective, not from an administrator account.

Evidence:

- [`iam-user-lab4-junior-analyst-created.png`](../screenshots/iam-user-lab4-junior-analyst-created.png)

## Step 4: Attach the Initial Over-Permissioned Policy

The test user was initially attached to the AWS managed policy:

```text
AmazonS3FullAccess
```

This was intentionally over-permissioned.

The purpose was to simulate a common IAM misconfiguration where a user receives broader access than required for their role.

The intended role only required read access to one assigned bucket, but `AmazonS3FullAccess` allowed broad S3 actions across the account.

Evidence:

- [`initial-overpermissive-policy-attached.png`](../screenshots/initial-overpermissive-policy-attached.png)

## Step 5: Configure the AWS CLI Profile

The AWS CLI was configured with a dedicated profile for the test user:

```text
lab4-junior-analyst
```

All access tests were run with:

```powershell
--profile lab4-junior-analyst
```

This ensured that the commands were executed from the perspective of the restricted IAM user.

## Step 6: Verify CLI Identity

Before testing permissions, the CLI identity was verified.

```powershell
aws sts get-caller-identity --profile lab4-junior-analyst
```

Expected identity:

```text
arn:aws:iam::211125298316:user/lab4-junior-analyst
```

This confirmed that the AWS CLI was using the intended IAM user and not an administrator account.

Evidence:

- [`aws-cli-get-caller-identity.png`](../screenshots/aws-cli-get-caller-identity.png)

## Step 7: Validate Baseline Access Before Remediation

Before remediation, the user was tested with the broad `AmazonS3FullAccess` policy attached.

### List S3 buckets

```powershell
aws s3 ls --profile lab4-junior-analyst
```

Result:

```text
Allowed
```

Security meaning:

The user could enumerate S3 buckets in the account, which was broader access than the intended role required.

Evidence:

- [`s3-access-allowed-before-remediation.png`](../screenshots/s3-access-allowed-before-remediation.png)

### Upload an object

```powershell
aws s3 cp lab4-test-file.txt s3://jw-lab4-iam-test-bucket-2026/ --profile lab4-junior-analyst
```

Result:

```text
Allowed
```

Security meaning:

The user could write objects to S3, even though the intended role only required read access.

Evidence:

- [`s3-object-upload-allowed-before-remediation.png`](../screenshots/s3-object-upload-allowed-before-remediation.png)

## Step 8: Create the Least-Privilege Policy

A custom least-privilege policy was created to replace the broad managed policy.

The new policy allowed only:

- `s3:ListBucket` on the assigned lab bucket
- `s3:GetObject` on objects inside the assigned lab bucket

Policy file:

- [`least-privilege-s3-read-policy.json`](../policies/least-privilege-s3-read-policy.json)

Evidence:

- [`least-privilege-policy-created.png`](../screenshots/least-privilege-policy-created.png)

## Step 9: Attach the Least-Privilege Policy

The custom least-privilege policy was attached to the `lab4-junior-analyst` user.

The broad `AmazonS3FullAccess` policy was removed so the user would only retain the scoped read-only permissions required for the role.

Evidence:

- [`least-privilege-policy-attached.png`](../screenshots/least-privilege-policy-attached.png)

## Step 10: Confirm Required Access Still Worked

After remediation, the user was still able to access the assigned bucket and read the assigned object.

Example command:

```powershell
aws s3 ls s3://jw-lab4-iam-test-bucket-2026 --profile lab4-junior-analyst
```

Result:

```text
Allowed
```

Security meaning:

The least-privilege policy preserved the user's required job function.

Evidence:

- [`s3-read-access-allowed-after-remediation.png`](../screenshots/s3-read-access-allowed-after-remediation.png)

## Step 11: Confirm Unnecessary Access Was Denied

After remediation, actions outside the user's intended role were denied.

Denied S3 actions included:

- Listing all S3 buckets
- Uploading objects
- Deleting objects
- Modifying the bucket policy

Denied IAM actions included:

- Listing IAM users
- Attaching `AdministratorAccess`
- Creating a new access key

Evidence:

- [`s3-list-all-buckets-denied-after-remediation.png`](../screenshots/s3-list-all-buckets-denied-after-remediation.png)
- [`s3-upload-denied-after-remediation.png`](../screenshots/s3-upload-denied-after-remediation.png)
- [`iam-list-users-denied-after-remediation.png`](../screenshots/iam-list-users-denied-after-remediation.png)
- [`session3-s3-delete-denied.png`](../screenshots/session3-s3-delete-denied.png)
- [`session3-s3-put-bucket-policy-denied.png`](../screenshots/session3-s3-put-bucket-policy-denied.png)
- [`session3-iam-attach-admin-policy-denied.png`](../screenshots/session3-iam-attach-admin-policy-denied.png)
- [`session3-iam-create-access-key-denied.png`](../screenshots/session3-iam-create-access-key-denied.png)

## Step 12: Verify CloudTrail Audit Evidence

CloudTrail was used to confirm that AWS recorded denied S3 and IAM activity from the test user.

CloudTrail provided audit evidence for events such as:

- `ListUsers`
- `AttachUserPolicy`
- `CreateAccessKey`
- `PutBucketPolicy`

These events showed that the actions were attempted by the `lab4-junior-analyst` user and denied by AWS policy enforcement.

Evidence:

- [`cloudtrail-session4-user-filtered-events.png`](../screenshots/cloudtrail-session4-user-filtered-events.png)
- [`cloudtrail-iam-list-users-access-denied.png`](../screenshots/cloudtrail-iam-list-users-access-denied.png)
- [`cloudtrail-iam-attach-admin-policy-access-denied.png`](../screenshots/cloudtrail-iam-attach-admin-policy-access-denied.png)
- [`cloudtrail-iam-create-access-key-access-denied.png`](../screenshots/cloudtrail-iam-create-access-key-access-denied.png)
- [`cloudtrail-s3-put-bucket-policy-access-denied.png`](../screenshots/cloudtrail-s3-put-bucket-policy-access-denied.png)

## Build Validation Checklist

| Build Item | Status | Evidence |
|---|---:|---|
| S3 lab bucket created | Complete | [`s3-lab-bucket-created.png`](../screenshots/s3-lab-bucket-created.png) |
| Test object uploaded | Complete | [`s3-test-object-uploaded.png`](../screenshots/s3-test-object-uploaded.png) |
| IAM test user created | Complete | [`iam-user-lab4-junior-analyst-created.png`](../screenshots/iam-user-lab4-junior-analyst-created.png) |
| Initial over-permissive policy attached | Complete | [`initial-overpermissive-policy-attached.png`](../screenshots/initial-overpermissive-policy-attached.png) |
| AWS CLI profile validated | Complete | [`aws-cli-get-caller-identity.png`](../screenshots/aws-cli-get-caller-identity.png) |
| Baseline broad S3 access confirmed | Complete | [`s3-access-allowed-before-remediation.png`](../screenshots/s3-access-allowed-before-remediation.png) |
| Baseline S3 upload confirmed | Complete | [`s3-object-upload-allowed-before-remediation.png`](../screenshots/s3-object-upload-allowed-before-remediation.png) |
| Least-privilege policy created | Complete | [`least-privilege-policy-created.png`](../screenshots/least-privilege-policy-created.png) |
| Least-privilege policy attached | Complete | [`least-privilege-policy-attached.png`](../screenshots/least-privilege-policy-attached.png) |
| Required read access preserved | Complete | [`s3-read-access-allowed-after-remediation.png`](../screenshots/s3-read-access-allowed-after-remediation.png) |
| Unnecessary S3 access denied | Complete | [`session3-s3-upload-denied.png`](../screenshots/session3-s3-upload-denied.png) |
| Risky IAM actions denied | Complete | [`session3-iam-attach-admin-policy-denied.png`](../screenshots/session3-iam-attach-admin-policy-denied.png) |
| CloudTrail evidence collected | Complete | [`cloudtrail-session4-user-filtered-events.png`](../screenshots/cloudtrail-session4-user-filtered-events.png) |

## Final Build Outcome

By the end of the setup and build phase, the lab environment included:

- A dedicated IAM test user
- A dedicated S3 lab bucket
- A test object for access validation
- An intentionally broad starting policy
- A custom least-privilege replacement policy
- AWS CLI testing from the restricted user's perspective
- CloudTrail audit evidence for denied activity

This setup created the foundation for validating IAM least privilege through before-and-after access testing.

The environment showed that broad permissions could be identified, reduced, and validated with real AWS evidence.
