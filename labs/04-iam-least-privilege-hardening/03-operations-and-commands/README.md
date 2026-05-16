# Operations and Commands

This section documents the AWS CLI commands used to validate identity, test S3 access, test denied IAM actions, and verify CloudTrail audit evidence.

All commands were run using the dedicated lab profile:

```powershell
--profile lab4-junior-analyst
```

This ensured the tests were performed from the perspective of the restricted IAM user, not from an administrator account.

## 1. Identity Verification

Before testing permissions, I confirmed which AWS identity the CLI profile was using.

```powershell
aws sts get-caller-identity --profile lab4-junior-analyst
```

Expected result:

```text
arn:aws:iam::211125298316:user/lab4-junior-analyst
```

Purpose:

- Confirm the AWS CLI was using the intended IAM user.
- Prevent accidentally testing with an administrator or personal account.
- Establish a clean baseline before access validation.

Evidence:

- `../screenshots/aws-cli-get-caller-identity.png`

## 2. Baseline S3 Access Before Remediation

The IAM user initially had the AWS managed `AmazonS3FullAccess` policy attached. This was intentionally over-permissioned to simulate a common cloud misconfiguration.

### List all buckets

```powershell
aws s3 ls --profile lab4-junior-analyst
```

Purpose:

- Test whether the user could enumerate S3 buckets across the account.
- Validate that the initial policy granted broader access than the role required.

Result:

```text
Allowed
```

Evidence:

- `../screenshots/s3-access-allowed-before-remediation.png`

### Upload object to lab bucket

```powershell
echo "This is a harmless test file for Lab 4 IAM least-privilege access testing." > lab4-test-file.txt
aws s3 cp lab4-test-file.txt s3://jw-lab4-iam-test-bucket-2026/ --profile lab4-junior-analyst
```

Purpose:

- Test whether the user could write objects to S3.
- Demonstrate that the original permissions allowed actions beyond the intended read-only role.

Result:

```text
Allowed
```

Evidence:

- `../screenshots/s3-object-upload-allowed-before-remediation.png`

## 3. Access Validation After Least-Privilege Remediation

After replacing broad S3 access with a scoped read-only policy, I retested both intended and risky actions.

### Confirm assigned bucket read access

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

Evidence:

- `../screenshots/s3-read-access-allowed-after-remediation.png`
- `../screenshots/session3-s3-read-access-allowed.png`

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

## 4. Denied S3 Actions After Remediation

### List all buckets

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

Evidence:

- `../screenshots/s3-list-all-buckets-denied-after-remediation.png`
- `../screenshots/session3-s3-list-all-buckets-denied.png`

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

Evidence:

- `../screenshots/s3-upload-denied-after-remediation.png`
- `../screenshots/session3-s3-upload-denied.png`

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

Evidence:

- `../screenshots/session3-s3-delete-denied.png`

### Modify bucket policy

A local JSON file was created to simulate an attempted bucket policy change:

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

Evidence:

- `../screenshots/session3-s3-put-bucket-policy-denied.png`
- `../screenshots/cloudtrail-s3-put-bucket-policy-access-denied.png`

## 5. Denied IAM Actions After Remediation

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

Evidence:

- `../screenshots/iam-list-users-denied-after-remediation.png`
- `../screenshots/session3-iam-list-users-denied.png`
- `../screenshots/cloudtrail-iam-list-users-access-denied.png`

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

Evidence:

- `../screenshots/session3-iam-attach-admin-policy-denied.png`
- `../screenshots/cloudtrail-iam-attach-admin-policy-access-denied.png`

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
- Confirm the user could not create additional credentials for itself.

Evidence:

- `../screenshots/session3-iam-create-access-key-denied.png`
- `../screenshots/cloudtrail-iam-create-access-key-access-denied.png`

## 6. CloudTrail Lookup

CloudTrail Event history was used to verify that AWS recorded the denied IAM and S3 actions.

Optional CLI lookup:

```powershell
aws cloudtrail lookup-events --lookup-attributes AttributeKey=Username,AttributeValue=lab4-junior-analyst --max-results 10
```

Purpose:

- Validate that the activity was not only blocked in the CLI, but also recorded in AWS audit logs.
- Confirm that denied actions produced CloudTrail evidence with event name, user identity, source IP address, event time, event source, and error code.

Evidence:

- `../screenshots/cloudtrail-session4-user-filtered-events.png`
- `../screenshots/cloudtrail-iam-list-users-access-denied.png`
- `../screenshots/cloudtrail-iam-attach-admin-policy-access-denied.png`
- `../screenshots/cloudtrail-iam-create-access-key-access-denied.png`
- `../screenshots/cloudtrail-s3-put-bucket-policy-access-denied.png`

## Final Validation Summary

| Test | Result After Remediation | Security Meaning |
|---|---|---|
| Read assigned S3 bucket | Allowed | Preserved required job access |
| Download assigned object | Allowed | Confirmed intended read-only access |
| List all S3 buckets | Denied | Reduced account-level reconnaissance |
| Upload object | Denied | Prevented unauthorized writes |
| Delete object | Denied | Prevented destructive object actions |
| Modify bucket policy | Denied | Prevented weakening resource-based access controls |
| List IAM users | Denied | Prevented IAM enumeration |
| Attach AdministratorAccess | Denied | Prevented privilege escalation |
| Create access key | Denied | Prevented credential persistence |
