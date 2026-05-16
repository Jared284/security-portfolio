# Reflections and Improvements

## Purpose

This section summarizes what I learned from the AWS IAM least-privilege hardening lab and how the design could be improved in a real AWS environment.

The lab focused on one core security idea:

```text
Least privilege should be tested, not assumed.
```

## What I Learned

This lab helped me understand IAM least privilege as more than just a best practice.

It showed how permissions directly control what an identity can do inside an AWS account.

The most important lesson was that an IAM policy should be validated through actual testing, not assumed to be correct just because it looks scoped in the console.

I validated the access-control design by testing both:

- Actions the user should be able to perform
- Actions the user should not be able to perform

This made the remediation more defensible because the final result was proven with:

- AWS CLI output
- S3 access tests
- IAM denied-action tests
- CloudTrail audit evidence
- Screenshot documentation

## Lab Summary

The lab followed this workflow:

```text
Create IAM user
→ attach broad S3 permissions
→ validate excessive access
→ replace broad access with least privilege
→ test allowed actions
→ test denied actions
→ review CloudTrail evidence
→ document hardening recommendations
```

The test identity was:

```text
lab4-junior-analyst
```

The assigned resource was:

```text
jw-lab4-iam-test-bucket-2026
```

The final result was a user that could read from one assigned S3 bucket, but could not perform unnecessary S3 or IAM actions.

## Why Over-Permissioned Identities Are Dangerous

The initial `AmazonS3FullAccess` policy gave the `lab4-junior-analyst` user more access than required.

That matters because if an access key is leaked or misused, the attacker inherits the permissions attached to that identity.

The risk is not only whether the user is malicious.

The risk is that valid credentials can be abused if the identity has unnecessary permissions.

A user with broader permissions creates a larger blast radius.

In this lab, broad S3 access meant the user could initially perform actions beyond the intended read-only role, including listing S3 buckets and uploading objects.

Evidence:

- [`s3-access-allowed-before-remediation.png`](../screenshots/s3-access-allowed-before-remediation.png)
- [`s3-object-upload-allowed-before-remediation.png`](../screenshots/s3-object-upload-allowed-before-remediation.png)

## Why Least Privilege Matters

Least privilege reduces risk by giving an identity only the permissions required for its job function.

In this lab, the intended job function was simple:

```text
Read from one assigned S3 bucket
```

The final policy allowed only:

- `s3:ListBucket`
- `s3:GetObject`

It did not allow:

- Listing all S3 buckets
- Uploading objects
- Deleting objects
- Modifying bucket policies
- Listing IAM users
- Attaching administrator permissions
- Creating new access keys

The key lesson is that a least-privilege policy should preserve legitimate access while removing unnecessary access.

Evidence:

- [`least-privilege-policy-created.png`](../screenshots/least-privilege-policy-created.png)
- [`least-privilege-policy-attached.png`](../screenshots/least-privilege-policy-attached.png)
- [`s3-read-access-allowed-after-remediation.png`](../screenshots/s3-read-access-allowed-after-remediation.png)

## Why Denied-Action Testing Matters

A policy is only useful if it behaves correctly when tested.

This lab did not stop after attaching the least-privilege policy. I tested actions that should have been denied.

Denied tests included:

| Action | Security Meaning |
|---|---|
| List all S3 buckets | Prevents broad S3 reconnaissance |
| Upload object | Prevents unauthorized writes |
| Delete object | Prevents destructive object actions |
| Modify bucket policy | Prevents weakening resource-based permissions |
| List IAM users | Prevents identity enumeration |
| Attach `AdministratorAccess` | Prevents privilege escalation |
| Create new access key | Prevents credential persistence |

This testing proved that the policy was not only smaller, but actually effective.

Evidence:

- [`s3-list-all-buckets-denied-after-remediation.png`](../screenshots/s3-list-all-buckets-denied-after-remediation.png)
- [`s3-upload-denied-after-remediation.png`](../screenshots/s3-upload-denied-after-remediation.png)
- [`session3-s3-delete-denied.png`](../screenshots/session3-s3-delete-denied.png)
- [`session3-s3-put-bucket-policy-denied.png`](../screenshots/session3-s3-put-bucket-policy-denied.png)
- [`iam-list-users-denied-after-remediation.png`](../screenshots/iam-list-users-denied-after-remediation.png)
- [`session3-iam-attach-admin-policy-denied.png`](../screenshots/session3-iam-attach-admin-policy-denied.png)
- [`session3-iam-create-access-key-denied.png`](../screenshots/session3-iam-create-access-key-denied.png)

## Why CloudTrail Matters

CloudTrail provided an audit trail of denied activity.

The AWS CLI showed that AWS denied the risky actions, but CloudTrail showed that those attempts were also recorded.

CloudTrail evidence included:

- Event name
- User identity
- Event source
- Source IP address
- Event time
- Error code

This is important because security teams need evidence after an access-control event.

CloudTrail helps answer questions like:

- Which identity attempted the action?
- What action was attempted?
- When did it happen?
- Where did the request come from?
- Was it allowed or denied?

Evidence:

- [`cloudtrail-session4-user-filtered-events.png`](../screenshots/cloudtrail-session4-user-filtered-events.png)
- [`cloudtrail-iam-list-users-access-denied.png`](../screenshots/cloudtrail-iam-list-users-access-denied.png)
- [`cloudtrail-iam-attach-admin-policy-access-denied.png`](../screenshots/cloudtrail-iam-attach-admin-policy-access-denied.png)
- [`cloudtrail-iam-create-access-key-access-denied.png`](../screenshots/cloudtrail-iam-create-access-key-access-denied.png)
- [`cloudtrail-s3-put-bucket-policy-access-denied.png`](../screenshots/cloudtrail-s3-put-bucket-policy-access-denied.png)

## Skills Practiced

This lab reinforced several practical cloud security skills:

- IAM policy analysis
- Least-privilege policy design
- AWS CLI access testing
- S3 permission validation
- IAM denied-action testing
- Privilege escalation testing
- Credential persistence testing
- CloudTrail audit review
- Security documentation
- Before-and-after remediation validation

## What Went Well

The strongest part of this lab was the full validation workflow.

The lab did not only show a policy change. It showed:

```text
Before state
→ risk identified
→ remediation applied
→ required access preserved
→ risky access denied
→ CloudTrail evidence collected
```

This made the project stronger because every major claim was backed by evidence.

## What I Would Improve in a Real Environment

### Use IAM roles instead of long-lived IAM user credentials

In production, I would avoid long-lived IAM user access keys when possible.

A better design would use IAM roles and temporary credentials, especially for workloads, automation, and cloud applications.

Temporary credentials reduce the risk of long-term access key exposure.

### Use IAM groups, roles, or identity federation

Instead of attaching policies directly to one user, I would manage access through IAM groups, roles, or identity federation.

This scales better and makes permission management cleaner.

For a real organization, direct user policy attachments can become hard to audit and maintain over time.

### Require MFA for sensitive operations

Sensitive actions should require stronger authentication controls.

Examples include:

- IAM policy changes
- Access key creation
- Bucket policy updates
- Role assumption into privileged accounts

MFA adds an extra layer of protection if credentials are compromised.

### Monitor IAM and S3 changes with alerts

CloudTrail recorded the denied activity, but a production environment should also generate alerts for high-risk events.

Examples of events worth monitoring:

- `AttachUserPolicy`
- `CreateAccessKey`
- `PutBucketPolicy`
- `DeleteBucketPolicy`
- `PutUserPolicy`
- `CreatePolicyVersion`
- `UpdateAssumeRolePolicy`

These events could be routed through:

- EventBridge
- CloudWatch
- Security Hub
- A SIEM

### Enable S3 data events where needed

CloudTrail management events captured IAM and bucket-policy activity.

In a production environment, S3 data events could also be enabled for sensitive buckets to monitor object-level activity such as:

- `GetObject`
- `PutObject`
- `DeleteObject`

This would provide deeper visibility into object-level access.

### Use IAM Access Analyzer

IAM Access Analyzer could help identify policies that grant broad or unintended access.

This would be useful for continuously reviewing whether identities have permissions beyond their intended role.

### Apply permission boundaries

Permission boundaries can limit the maximum permissions an identity can receive.

This would add another layer of protection against accidental or unauthorized privilege expansion.

For example, even if someone tried to attach a broader policy, the permission boundary could prevent the identity from exceeding its approved access level.

### Rotate or remove unused access keys

Access keys should be reviewed regularly.

Unused keys should be removed, and active keys should be rotated according to organizational policy.

In production, I would also monitor for access keys that have not been used recently.

### Add automated remediation

A more advanced version of this lab could include automated response.

For example:

```text
CloudTrail event
→ EventBridge rule
→ Lambda function
→ alert or disable risky access
```

This would turn the lab from manual validation into a stronger detection-and-response workflow.

## Future Lab Improvements

If I expanded this project, I would add:

- IAM Access Analyzer findings
- EventBridge alerts for denied IAM actions
- CloudWatch metrics for risky IAM API calls
- S3 data event logging for object-level access
- Permission boundary testing
- MFA condition-key policy examples
- A cleanup script to remove lab resources safely
- A diagram showing before-and-after IAM permissions

## Final Reflection

This lab showed a complete IAM hardening workflow:

```text
Over-permissioned identity
→ risky access validated
→ least-privilege policy applied
→ allowed and denied actions tested
→ CloudTrail evidence reviewed
→ hardening recommendations documented
```

The main takeaway is that least privilege is not just about writing a smaller policy.

It is about proving that required access still works while unnecessary and risky access is removed.

This project helped me understand how IAM policy design, CLI validation, and CloudTrail evidence work together in real cloud security operations.
