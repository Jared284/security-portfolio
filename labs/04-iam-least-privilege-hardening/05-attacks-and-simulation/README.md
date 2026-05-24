# 05 - Attacks and Simulation

## Purpose

This section documents the controlled misuse testing performed in the AWS IAM Least Privilege and Access Control Hardening Lab.

The goal was not to attack AWS. The goal was to safely simulate what an over-permissioned or misused cloud identity could attempt, then validate that risky actions were denied after least-privilege remediation.

This section focuses on the security-testing side of the lab:

```text
What could this identity do before remediation?
        ↓
What should this identity still be allowed to do?
        ↓
What should this identity be blocked from doing?
        ↓
Did AWS enforce those restrictions?
        ↓
Did CloudTrail record the attempts?
```

---

## Simulation Scope

The test identity was:

```text
lab4-junior-analyst
```

The assigned resource was:

```text
jw-lab4-iam-test-bucket-2026
```

The simulation focused on four categories of activity:

1. Required access
2. Unnecessary S3 access
3. S3 access-control modification
4. Risky IAM actions

---

## Why These Tests Matter

Over-permissioned IAM identities are dangerous because attackers often do not need to exploit a software vulnerability if they can misuse valid credentials.

If an access key is exposed or stolen, the attacker inherits the permissions attached to that identity.

That means the real security question is not only:

```text
Can someone authenticate?
```

It is also:

```text
What can this identity do after it authenticates?
```

This lab tested whether the `lab4-junior-analyst` user could perform actions outside its intended role.

---

## Threat Model

The simulated threat was credential misuse.

```text
Assumption:
The lab4-junior-analyst access key is used by someone who should not have broad AWS access.

Question:
How much damage, reconnaissance, or persistence could that identity perform?

Goal:
Reduce the identity's blast radius by removing unnecessary permissions.
```

This is why the tests included actions related to:

- Reconnaissance
- Unauthorized writes
- Destructive actions
- Access-control changes
- Privilege escalation
- Credential persistence

---

## Before Remediation: Over-Permissioned Starting Point

Before remediation, the user had the AWS managed policy:

```text
AmazonS3FullAccess
```

This allowed broader S3 access than the user needed.

Baseline evidence showed that the user could:

- List S3 buckets in the account
- Upload an object to the lab bucket

This proved that the starting permission set was too broad for a user that only needed read access to one assigned S3 bucket.

### Evidence

- [`s3-access-allowed-before-remediation.png`](../screenshots/s3-access-allowed-before-remediation.png)
- [`s3-object-upload-allowed-before-remediation.png`](../screenshots/s3-object-upload-allowed-before-remediation.png)

---

## Required Access Test

After remediation, the user was expected to retain read-only access to the assigned lab bucket.

Tested actions:

- List the assigned S3 bucket
- Download the assigned test object

Result:

```text
Allowed
```

### Security Meaning

The remediation did not break the user's required access.

The user could still perform the intended read-only task, which proved that least privilege preserved business functionality while removing unnecessary permissions.

### Evidence

- [`s3-read-access-allowed-after-remediation.png`](../screenshots/s3-read-access-allowed-after-remediation.png)
- [`session3-s3-read-access-allowed.png`](../screenshots/session3-s3-read-access-allowed.png)

---

## Unnecessary S3 Action Tests

After remediation, the user attempted multiple S3 actions that were not required for the role.

---

### Test 1: List All S3 Buckets

Result:

```text
AccessDenied
```

### Security Meaning

This prevented broad account-level S3 enumeration.

The user could not discover unrelated storage resources, which reduced the reconnaissance value of the credentials if they were misused.

### Evidence

- [`s3-list-all-buckets-denied-after-remediation.png`](../screenshots/s3-list-all-buckets-denied-after-remediation.png)
- [`session3-s3-list-all-buckets-denied.png`](../screenshots/session3-s3-list-all-buckets-denied.png)

---

### Test 2: Upload an Object

Result:

```text
AccessDenied
```

### Security Meaning

This prevented unauthorized writes to the bucket.

The user could no longer modify bucket contents by uploading new objects, which enforced the intended read-only role.

### Evidence

- [`s3-upload-denied-after-remediation.png`](../screenshots/s3-upload-denied-after-remediation.png)
- [`session3-s3-upload-denied.png`](../screenshots/session3-s3-upload-denied.png)

---

### Test 3: Delete an Object

Result:

```text
AccessDenied
```

### Security Meaning

This prevented destructive object actions.

The user could not delete objects from the bucket, reducing the risk of accidental or malicious data loss.

### Evidence

- [`session3-s3-delete-denied.png`](../screenshots/session3-s3-delete-denied.png)

---

### Test 4: Modify the Bucket Policy

Result:

```text
AccessDenied
```

### Security Meaning

This prevented the user from weakening bucket-level access controls.

The user could not modify the bucket policy to expose data, grant broader access, or alter the resource's security boundary.

### Evidence

- [`session3-s3-put-bucket-policy-denied.png`](../screenshots/session3-s3-put-bucket-policy-denied.png)
- [`cloudtrail-s3-put-bucket-policy-access-denied.png`](../screenshots/cloudtrail-s3-put-bucket-policy-access-denied.png)

---

## Risky IAM Action Tests

The user also attempted IAM actions that could support reconnaissance, privilege escalation, or credential persistence.

These tests were important because identity-based attacks in cloud environments often involve discovering users, attaching stronger policies, or creating new credentials.

---

### Test 5: List IAM Users

Result:

```text
AccessDenied
```

### Security Meaning

This prevented IAM enumeration.

The user could not list IAM users in the account, which limited identity reconnaissance if the credentials were misused.

### Evidence

- [`iam-list-users-denied-after-remediation.png`](../screenshots/iam-list-users-denied-after-remediation.png)
- [`cloudtrail-iam-list-users-access-denied.png`](../screenshots/cloudtrail-iam-list-users-access-denied.png)

---

### Test 6: Attach AdministratorAccess

Result:

```text
AccessDenied
```

### Security Meaning

This prevented privilege escalation.

The user could not attach `AdministratorAccess` to itself, which blocked a direct path from a restricted identity to administrator-level permissions.

### Evidence

- [`session3-iam-attach-admin-policy-denied.png`](../screenshots/session3-iam-attach-admin-policy-denied.png)
- [`cloudtrail-iam-attach-admin-policy-access-denied.png`](../screenshots/cloudtrail-iam-attach-admin-policy-access-denied.png)

---

### Test 7: Create New Access Key

Result:

```text
AccessDenied
```

### Security Meaning

This prevented credential persistence.

The user could not create an additional long-lived access key that could be used to maintain access.

### Evidence

- [`session3-iam-create-access-key-denied.png`](../screenshots/session3-iam-create-access-key-denied.png)
- [`cloudtrail-iam-create-access-key-access-denied.png`](../screenshots/cloudtrail-iam-create-access-key-access-denied.png)

---

## Simulation Results

| Category | Action | Result After Remediation | Security Purpose |
|---|---|---:|---|
| Required access | Read assigned S3 bucket | Allowed | Preserve intended access |
| Required access | Download assigned object | Allowed | Preserve intended object read access |
| S3 misuse | List all buckets | Denied | Prevent account-wide reconnaissance |
| S3 misuse | Upload object | Denied | Prevent unauthorized writes |
| S3 misuse | Delete object | Denied | Prevent destructive actions |
| S3 misuse | Modify bucket policy | Denied | Prevent access-control weakening |
| IAM misuse | List IAM users | Denied | Prevent identity enumeration |
| IAM misuse | Attach `AdministratorAccess` | Denied | Prevent privilege escalation |
| IAM misuse | Create access key | Denied | Prevent credential persistence |

---

## CloudTrail Evidence

The denied activity was also validated in CloudTrail.

CloudTrail confirmed that attempted IAM and S3 actions were logged with `AccessDenied` error codes. This provided audit evidence that the activity occurred and that AWS policy enforcement blocked it.

Relevant CloudTrail screenshots:

- [`cloudtrail-session4-user-filtered-events.png`](../screenshots/cloudtrail-session4-user-filtered-events.png)
- [`cloudtrail-iam-list-users-access-denied.png`](../screenshots/cloudtrail-iam-list-users-access-denied.png)
- [`cloudtrail-iam-attach-admin-policy-access-denied.png`](../screenshots/cloudtrail-iam-attach-admin-policy-access-denied.png)
- [`cloudtrail-iam-create-access-key-access-denied.png`](../screenshots/cloudtrail-iam-create-access-key-access-denied.png)
- [`cloudtrail-s3-put-bucket-policy-access-denied.png`](../screenshots/cloudtrail-s3-put-bucket-policy-access-denied.png)

---

## What This Simulation Proved

This simulation proved that least privilege was enforced in practice.

The remediated IAM user could still perform the intended read-only S3 task, but could not perform higher-risk actions outside its role.

The test confirmed that the user could not:

- Enumerate all S3 buckets
- Upload new S3 objects
- Delete S3 objects
- Modify S3 bucket policies
- Enumerate IAM users
- Attach administrator permissions
- Create additional access keys

---

## Security Takeaway

The most important takeaway is that IAM hardening should be validated through actual access testing.

A policy may look correct in the console, but the real test is whether the identity can perform only the actions it should be able to perform.

This lab validated that the identity's blast radius was reduced by removing unnecessary permissions and confirming that risky actions returned `AccessDenied`.

It also confirmed that denied actions were recorded in CloudTrail, which makes the hardening work auditable and defensible.
