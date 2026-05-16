# Attacks and Simulation

This section documents the controlled misuse testing performed in the Lab 4 AWS environment.

The goal was not to attack AWS. The goal was to safely test what an over-permissioned or misused cloud identity could attempt, then validate that risky actions were denied after least-privilege remediation.

## Simulation Scope

The test identity was:

```text
lab4-junior-analyst
```

The assigned resource was:

```text
jw-lab4-iam-test-bucket-2026
```

The simulation focused on three categories of activity:

1. Intended access
2. Unnecessary S3 actions
3. Risky IAM actions

## Why These Tests Matter

Over-permissioned IAM identities are dangerous because attackers often do not need a software vulnerability if they can misuse valid credentials.

If an access key is exposed or stolen, the attacker inherits the permissions attached to that identity. This makes least privilege important because it limits what the identity can do even if the credentials are misused.

This lab tested whether the `lab4-junior-analyst` identity could perform actions outside its intended role.

## Intended Access Test

The user was expected to retain read-only access to the assigned lab bucket.

Tested actions:

- List the assigned S3 bucket
- Download the assigned test object

Result:

```text
Allowed
```

Security meaning:

- The remediation did not break the user's required access.
- The user could still perform the intended read-only task.

Evidence:

- `../screenshots/s3-read-access-allowed-after-remediation.png`
- `../screenshots/session3-s3-read-access-allowed.png`

## Unnecessary S3 Action Tests

After remediation, the user attempted multiple S3 actions that were not required for the role.

### List all S3 buckets

Result:

```text
AccessDenied
```

Security meaning:

- Prevents broad account-level S3 enumeration.
- Limits the user's ability to discover unrelated storage resources.

Evidence:

- `../screenshots/s3-list-all-buckets-denied-after-remediation.png`
- `../screenshots/session3-s3-list-all-buckets-denied.png`

### Upload an object

Result:

```text
AccessDenied
```

Security meaning:

- Prevents unauthorized writes to the bucket.
- Reduces risk of malicious or accidental object modification.

Evidence:

- `../screenshots/s3-upload-denied-after-remediation.png`
- `../screenshots/session3-s3-upload-denied.png`

### Delete an object

Result:

```text
AccessDenied
```

Security meaning:

- Prevents destructive object actions.
- Protects bucket contents from unauthorized deletion.

Evidence:

- `../screenshots/session3-s3-delete-denied.png`

### Modify the bucket policy

Result:

```text
AccessDenied
```

Security meaning:

- Prevents the user from weakening bucket-level access controls.
- Protects resource-based permissions from unauthorized changes.

Evidence:

- `../screenshots/session3-s3-put-bucket-policy-denied.png`
- `../screenshots/cloudtrail-s3-put-bucket-policy-access-denied.png`

## Risky IAM Action Tests

The user also attempted IAM actions that could support privilege escalation, reconnaissance, or persistence.

### List IAM users

Result:

```text
AccessDenied
```

Security meaning:

- Prevents IAM enumeration.
- Limits visibility into account identities.

Evidence:

- `../screenshots/iam-list-users-denied-after-remediation.png`
- `../screenshots/session3-iam-list-users-denied.png`
- `../screenshots/cloudtrail-iam-list-users-access-denied.png`

### Attach AdministratorAccess

Result:

```text
AccessDenied
```

Security meaning:

- Prevents privilege escalation.
- Confirms the user cannot attach administrator permissions to itself.

Evidence:

- `../screenshots/session3-iam-attach-admin-policy-denied.png`
- `../screenshots/cloudtrail-iam-attach-admin-policy-access-denied.png`

### Create new access key

Result:

```text
AccessDenied
```

Security meaning:

- Prevents credential persistence.
- Confirms the user cannot create additional long-lived credentials.

Evidence:

- `../screenshots/session3-iam-create-access-key-denied.png`
- `../screenshots/cloudtrail-iam-create-access-key-access-denied.png`

## Simulation Results

| Category | Action | Result | Security Purpose |
|---|---|---|---|
| Intended access | Read assigned S3 bucket | Allowed | Preserve required access |
| S3 misuse | List all buckets | Denied | Prevent reconnaissance |
| S3 misuse | Upload object | Denied | Prevent unauthorized writes |
| S3 misuse | Delete object | Denied | Prevent destructive actions |
| S3 misuse | Modify bucket policy | Denied | Prevent access-control weakening |
| IAM misuse | List IAM users | Denied | Prevent identity enumeration |
| IAM misuse | Attach AdministratorAccess | Denied | Prevent privilege escalation |
| IAM misuse | Create access key | Denied | Prevent credential persistence |

## CloudTrail Evidence

The denied activity was also validated in CloudTrail.

CloudTrail confirmed that the attempted IAM and S3 actions were logged with `AccessDenied` error codes. This provides audit evidence that the activity occurred and that AWS enforcement blocked it.

Relevant CloudTrail screenshots:

- `../screenshots/cloudtrail-session4-user-filtered-events.png`
- `../screenshots/cloudtrail-iam-list-users-access-denied.png`
- `../screenshots/cloudtrail-iam-attach-admin-policy-access-denied.png`
- `../screenshots/cloudtrail-iam-create-access-key-access-denied.png`
- `../screenshots/cloudtrail-s3-put-bucket-policy-access-denied.png`

## Final Takeaway

The simulation showed that the remediated IAM user could still perform its intended read-only task, but could not perform higher-risk actions outside its role.

This validates the least-privilege design and demonstrates how access-control hardening can reduce blast radius if credentials are misused.
