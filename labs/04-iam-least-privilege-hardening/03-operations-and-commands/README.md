# Operations and Commands

## Purpose

This section documents the AWS CLI commands used to validate identity, test S3 permissions, verify least-privilege remediation, test denied IAM actions, and confirm CloudTrail audit evidence.

The goal was to test access from the perspective of the restricted IAM user, not from an administrator account.

All commands were run using the dedicated AWS CLI profile:

```powershell
--profile lab4-junior-analyst
```

## Command Flow

```text
Verify identity
→ test baseline broad S3 access
→ apply least-privilege remediation
→ retest required read access
→ test denied S3 actions
→ test denied IAM actions
→ verify CloudTrail audit evidence
```

## 1. Identity Verification

Before testing permissions, I confirmed which AWS identity the CLI profile was using.

```powershell
aws sts get-caller-identity --profile lab4-junior-analyst
```

Expected identity:

```text
arn:aws:iam::211125298316:user/lab4-junior-analyst
```

Purpose:

- Confirm the AWS CLI was using the intended IAM user.
- Prevent accidentally testing with an administrator account.
- Establish a clean baseline before access validation.

Evidence:

- [`aws-cli-get-caller-identity.png`](../screenshots/aws-cli-get-caller-identity.png)

## 2. Baseline S3 Access Before Remediation

The IAM user initially had the AWS managed `AmazonS3FullAccess` policy attached.

This was intentionally over-permissioned to simulate a common cloud misconfiguration.

### List all S3 buckets

```powershell
aws s3 ls --profile lab4-junior-analyst
```

Result:

```text
Allowed
```

Purpose:

- Test whether the user could enumerate S3 buckets across the account.
- Validate that the initial policy granted broader access than the role required.

Security meaning:

The user could view S3 buckets beyond the assigned lab bucket. This increased reconnaissance risk and showed that the starting permission set was too broad.

Evidence:

- [`s3-access-allowed-before-remediation.png`](../screenshots/s3-access-allowed-before-remediation.png)

### Upload object to the lab bucket

```powershell
echo "This is a harmless test file for Lab 4 IAM least-privilege access testing." > lab4-test-file.txt
aws s3 cp lab4-test-file.txt s3://jw-lab4-iam-test-bucket-2026/ --profile lab4-junior-analyst
```

Result:

```text
Allowed
```

Purpose:

- Test whether the user could write objects to S3.
- Demonstrate that the original permissions allowed actions beyond the intended read-only role.

Security meaning:

The user had unnecessary write access. For a read-only analyst role, upload capability increases risk because the identity can modify cloud storage contents.

Evidence:

- [`s3-object-upload-allowed-before-remediation.png`](../screenshots/s3-object-upload-allowed-before-remediation.png)

## 3. Least-Privilege Remediation

The broad `AmazonS3FullAccess` policy was removed and replaced with a custom scoped policy.

The remediated policy allowed only:

- `s3:ListBucket` on `jw-lab4-iam-test-bucket-2026`
- `s3:GetObject` on objects inside `jw-lab4-iam-test-bucket-2026`

Policy file:

- [`least-privilege-s3-read-policy.json`](../policies/least-privilege-s3-read-policy.json)

Evidence:

- [`least-privilege-policy-created.png`](../screenshots/least-privilege-policy-created.png)
- [`least-privilege-policy-attached.png`](../screenshots/least-privilege-policy-attached.png)

## 4. Required Access Validation After Remediation

After replacing broad S3 access with a scoped read-only policy, I retested the actions the user should still be able to perform.

### List assigned lab bucket

```powershell
aws s3 ls s3://jw-lab4-iam-test-bucket-2026 --profile lab4-junior-analyst
```

Result:

```text
Allowed
```

Purpose:

- Confirm the user could still perform its intended job function.
- Validate that least privilege preserved required access.

Security meaning:

The remediation did not break legitimate access. The user still had the minimum permissions needed for the assigned task.

Evidence:

- [`s3-read-access-allowed-after-remediation.png`](../screenshots/s3-read-access-allowed-after-remediation.png)
- [`session3-s3-read-access-allowed.png`](../screenshots/session3-s3-read-access-allowed.png)

### Download assigned object

```powershell
aws s3 cp s3://jw-lab4-iam-test-bucket-2026/lab4-test-file.txt downloaded-lab4-test-file-session3.txt --profile lab4-junior-analyst
```

Result:

```text
Allowed
```

Purpose:

- Confirm `s3:GetObject` worked for objects inside the assigned bucket.
- Validate that the custom read-only policy allowed intended object access.

## 5. Denied S3 Actions After Remediation

After confirming required access still worked, I tested S3 actions that should not be allowed for the user.

### List all S3 buckets

```powershell
aws s3 ls --profile lab4-junior-analyst
```

Result:

```text
AccessDenied
```

Purpose:

- Confirm the user could no longer enumerate all S3 buckets in the account.
- Reduce account-level reconnaissance risk.

Security meaning:

The user was restricted to the assigned bucket instead of having broad account-wide S3 visibility.

Evidence:

- [`s3-list-all-buckets-denied-after-remediation.png`](../screenshots/s3-list-all-buckets-denied-after-remediation.png)
- [`session3-s3-list-all-buckets-denied.png`](../screenshots/session3-s3-list-all-buckets-denied.png)

### Upload object

```powershell
echo "Session 3 upload test" > session3-upload-test.txt
aws s3 cp session3-upload-test.txt s3://jw-lab4-iam-test-bucket-2026/session3-upload-test.txt --profile lab4-junior-analyst
```

Result:

```text
AccessDenied
```

Purpose:

- Confirm the user could not write new objects after remediation.
- Validate that unnecessary write permissions were removed.

Security meaning:

The user could no longer modify bucket contents by uploading new objects.

Evidence:

- [`s3-upload-denied-after-remediation.png`](../screenshots/s3-upload-denied-after-remediation.png)
- [`session3-s3-upload-denied.png`](../screenshots/session3-s3-upload-denied.png)

### Delete object

```powershell
aws s3 rm s3://jw-lab4-iam-test-bucket-2026/lab4-test-file.txt --profile lab4-junior-analyst
```

Result:

```text
AccessDenied
```

Purpose:

- Confirm the user could not perform destructive object actions.
- Validate that delete access was not included in the scoped policy.

Security meaning:

The user could not delete objects from the bucket, reducing the risk of destructive misuse.

Evidence:

- [`session3-s3-delete-denied.png`](../screenshots/session3-s3-delete-denied.png)

### Modify bucket policy

A local JSON file was created to simulate an attempted bucket policy change.

```powershell
@'
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "FakePolicyTest",
      "Effect": "Allow",
      "Principal": "*",
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::jw-lab4-iam-test-bucket-2026/*"
    }
  ]
}
'@ > fake-bucket-policy.json
```

Then the restricted user attempted to apply it:

```powershell
aws s3api put-bucket-policy --bucket jw-lab4-iam-test-bucket-2026 --policy file://fake-bucket-policy.json --profile lab4-junior-analyst
```

Result:

```text
AccessDenied
```

Purpose:

- Confirm the user could not modify bucket-level access controls.
- Prevent the user from weakening resource-based permissions.

Security meaning:

The user could not change the bucket policy to grant broader access or weaken the resource's security boundary.

Evidence:

- [`session3-s3-put-bucket-policy-denied.png`](../screenshots/session3-s3-put-bucket-policy-denied.png)
- [`cloudtrail-s3-put-bucket-policy-access-denied.png`](../screenshots/cloudtrail-s3-put-bucket-policy-access-denied.png)

## 6. Denied IAM Actions After Remediation

The user also attempted IAM actions that could support reconnaissance, privilege escalation, or credential persistence.

### List IAM users

```powershell
aws iam list-users --profile lab4-junior-analyst
```

Result:

```text
AccessDenied
```

Purpose:

- Confirm the user could not enumerate IAM identities.
- Validate that the user had no unnecessary IAM visibility.

Security meaning:

The user could not list IAM users in the account, limiting identity reconnaissance.

Evidence:

- [`iam-list-users-denied-after-remediation.png`](../screenshots/iam-list-users-denied-after-remediation.png)
- [`cloudtrail-iam-list-users-access-denied.png`](../screenshots/cloudtrail-iam-list-users-access-denied.png)

### Attach AdministratorAccess

```powershell
aws iam attach-user-policy --user-name lab4-junior-analyst --policy-arn arn:aws:iam::aws:policy/AdministratorAccess --profile lab4-junior-analyst
```

Result:

```text
AccessDenied
```

Purpose:

- Simulate a privilege escalation attempt.
- Confirm the user could not attach administrator permissions to itself.

Security meaning:

The user could not escalate from a restricted identity to an administrator-level identity.

Evidence:

- [`session3-iam-attach-admin-policy-denied.png`](../screenshots/session3-iam-attach-admin-policy-denied.png)
- [`cloudtrail-iam-attach-admin-policy-access-denied.png`](../screenshots/cloudtrail-iam-attach-admin-policy-access-denied.png)

### Create new access key

```powershell
aws iam create-access-key --user-name lab4-junior-analyst --profile lab4-junior-analyst
```

Result:

```text
AccessDenied
```

Purpose:

- Simulate a credential persistence attempt.
- Confirm the user could not create additional long-lived credentials.

Security meaning:

The user could not create a new access key that could be used to maintain access.

Evidence:

- [`session3-iam-create-access-key-denied.png`](../screenshots/session3-iam-create-access-key-denied.png)
- [`cloudtrail-iam-create-access-key-access-denied.png`](../screenshots/cloudtrail-iam-create-access-key-access-denied.png)

## 7. CloudTrail Lookup and Audit Validation

CloudTrail Event history was used to verify that AWS recorded the denied IAM and S3 actions.

Optional CLI lookup:

```powershell
aws cloudtrail lookup-events --lookup-attributes AttributeKey=Username,AttributeValue=lab4-junior-analyst --max-results 10
```

Purpose:

- Validate that denied actions were not only blocked in the CLI, but also recorded in AWS audit logs.
- Confirm that CloudTrail captured evidence of user activity and policy enforcement.

CloudTrail evidence included:

- Event name
- User identity
- Event source
- Source IP address
- Event time
- Error code

Evidence:

- [`cloudtrail-session4-user-filtered-events.png`](../screenshots/cloudtrail-session4-user-filtered-events.png)
- [`cloudtrail-iam-list-users-access-denied.png`](../screenshots/cloudtrail-iam-list-users-access-denied.png)
- [`cloudtrail-iam-attach-admin-policy-access-denied.png`](../screenshots/cloudtrail-iam-attach-admin-policy-access-denied.png)
- [`cloudtrail-iam-create-access-key-access-denied.png`](../screenshots/cloudtrail-iam-create-access-key-access-denied.png)
- [`cloudtrail-s3-put-bucket-policy-access-denied.png`](../screenshots/cloudtrail-s3-put-bucket-policy-access-denied.png)

## Final Validation Matrix

| Test | Command Area | Result After Remediation | Security Meaning |
|---|---|---:|---|
| Verify identity | STS | Allowed | Confirmed correct IAM user was being tested |
| List assigned S3 bucket | S3 | Allowed | Preserved required job access |
| Download assigned object | S3 | Allowed | Confirmed intended read-only access |
| List all S3 buckets | S3 | Denied | Reduced account-level reconnaissance |
| Upload object | S3 | Denied | Prevented unauthorized writes |
| Delete object | S3 | Denied | Prevented destructive object actions |
| Modify bucket policy | S3 API | Denied | Prevented access-control weakening |
| List IAM users | IAM | Denied | Prevented identity enumeration |
| Attach `AdministratorAccess` | IAM | Denied | Prevented privilege escalation |
| Create access key | IAM | Denied | Prevented credential persistence |

## Key Takeaway

The AWS CLI tests proved that the `lab4-junior-analyst` user could still perform its required read-only S3 task, but could not perform unnecessary S3 actions or risky IAM actions.

This validated the least-privilege remediation with actual command output and CloudTrail audit evidence.
