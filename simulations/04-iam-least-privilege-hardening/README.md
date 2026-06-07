# AWS IAM Least Privilege and Access Control Hardening

## Overview

This lab demonstrates how to identify, reduce, and validate over-permissioned AWS IAM access.

I created a dedicated IAM test user, intentionally granted it broad S3 permissions, validated the excessive access through AWS CLI testing, replaced the broad access with a scoped least-privilege policy, and confirmed the remediation through allowed-action testing, denied-action testing, and CloudTrail audit evidence.

The lab follows a practical IAM hardening workflow:

```text
Over-permissioned identity
        ↓
Risky access validated
        ↓
Broad policy removed
        ↓
Least-privilege policy applied
        ↓
Required access preserved
        ↓
Unnecessary actions denied
        ↓
CloudTrail evidence reviewed
```

The goal was not only to write a smaller IAM policy, but to prove that the new access model worked in a real AWS environment.

---

## What This Lab Proves

This project demonstrates that I can:

- Identify over-permissioned IAM access
- Validate excessive permissions using the AWS CLI
- Replace broad managed policies with scoped custom policies
- Preserve required business access after remediation
- Deny unnecessary S3 actions
- Deny risky IAM actions
- Use CloudTrail to confirm denied activity
- Document least-privilege hardening with evidence

---

## Why I Built This Lab

IAM is one of the most important security layers in AWS.

A single over-permissioned identity can create major cloud security risk if it can access resources, modify policies, or escalate privileges beyond its intended job function.

This lab was built to show how least privilege can be applied and validated in a real AWS environment. The focus was not only on policy design, but on proof:

- What could the user do before remediation?
- What should the user still be able to do after remediation?
- What should the user no longer be able to do?
- Are denied actions visible in CloudTrail?

That makes the lab more realistic than simply writing an IAM policy on paper.

---

## Scenario

The lab simulates a junior analyst-style IAM identity that only needs read access to one assigned S3 bucket.

The user initially had excessive S3 access through the AWS managed `AmazonS3FullAccess` policy.

That broad policy was then replaced with a custom least-privilege policy that allowed only:

- `s3:ListBucket` on the assigned lab bucket
- `s3:GetObject` on objects inside the assigned lab bucket

After remediation, required read access still worked, while unnecessary and risky actions returned `AccessDenied`.

---

## Test Identity and Resource

```text
IAM User: lab4-junior-analyst
Assigned S3 Bucket: jw-lab4-iam-test-bucket-2026
Initial Policy: AmazonS3FullAccess
Remediated Policy: Custom least-privilege S3 read policy
Audit Layer: AWS CloudTrail
Testing Method: AWS CLI
```

---

## Security Concepts Demonstrated

- Identity and Access Management
- Least privilege
- IAM policy design
- Access control validation
- Cloud misconfiguration risk
- Blast radius reduction
- Privilege escalation prevention
- Credential persistence prevention
- S3 access control
- CloudTrail audit logging
- Allowed and denied API activity validation

---

## AWS Services and Tools Used

- AWS IAM
- Amazon S3
- AWS CloudTrail
- AWS CLI

---

## Lab Workflow

1. Created a dedicated IAM test user.
2. Created an assigned S3 lab bucket.
3. Uploaded a harmless test object.
4. Attached an intentionally broad S3 policy.
5. Validated baseline broad access through the AWS CLI.
6. Removed the broad managed policy.
7. Created and attached a custom least-privilege S3 read policy.
8. Retested required access after remediation.
9. Tested denied S3 and IAM actions.
10. Reviewed CloudTrail evidence for denied activity.
11. Documented findings, hardening decisions, and future improvements.

---

## Before Remediation

Before remediation, the `lab4-junior-analyst` user had the AWS managed `AmazonS3FullAccess` policy attached.

This allowed broader S3 access than the user's intended role required.

The user was able to:

- List S3 buckets in the account
- Upload an object to the lab S3 bucket

This represented unnecessary access because the intended role only required read access to one assigned S3 bucket.

### Evidence

- [`initial-overpermissive-policy-attached.png`](screenshots/initial-overpermissive-policy-attached.png)
- [`s3-access-allowed-before-remediation.png`](screenshots/s3-access-allowed-before-remediation.png)
- [`s3-object-upload-allowed-before-remediation.png`](screenshots/s3-object-upload-allowed-before-remediation.png)

---

## Remediation

The broad `AmazonS3FullAccess` policy was removed and replaced with a custom least-privilege policy.

The new policy allowed only the minimum permissions required for the test user's role.

| Permission | Resource | Purpose |
|---|---|---|
| `s3:ListBucket` | `arn:aws:s3:::jw-lab4-iam-test-bucket-2026` | Allow the user to list the assigned bucket |
| `s3:GetObject` | `arn:aws:s3:::jw-lab4-iam-test-bucket-2026/*` | Allow the user to read objects inside the assigned bucket |

Policy file:

- [`least-privilege-s3-read-policy.json`](policies/least-privilege-s3-read-policy.json)

### Evidence

- [`least-privilege-policy-created.png`](screenshots/least-privilege-policy-created.png)
- [`least-privilege-policy-attached.png`](screenshots/least-privilege-policy-attached.png)

---

## Validation Results

After remediation, I retested both required access and unnecessary access.

| Action Tested | Result After Remediation | Security Meaning |
|---|---:|---|
| List assigned S3 bucket | Allowed | Required read-only job access preserved |
| Download assigned object | Allowed | Required object read access preserved |
| List all S3 buckets | Denied | Prevented broad S3 enumeration |
| Upload object | Denied | Prevented unauthorized object writes |
| Delete object | Denied | Prevented destructive object actions |
| Modify bucket policy | Denied | Prevented access-control weakening |
| List IAM users | Denied | Prevented IAM identity enumeration |
| Attach `AdministratorAccess` | Denied | Prevented privilege escalation |
| Create new access key | Denied | Prevented credential persistence |

This validation confirmed that the remediation did not break required access, but did remove unnecessary and higher-risk permissions.

---

## CloudTrail Evidence

CloudTrail was used to confirm that denied S3 and IAM actions were recorded as audit events.

CloudTrail evidence showed:

- Event name
- User identity
- Event source
- Source IP address
- Event time
- `AccessDenied` error code

This matters because access control is stronger when denied activity can also be audited and investigated.

### Evidence

- [`cloudtrail-session4-user-filtered-events.png`](screenshots/cloudtrail-session4-user-filtered-events.png)
- [`cloudtrail-iam-list-users-access-denied.png`](screenshots/cloudtrail-iam-list-users-access-denied.png)
- [`cloudtrail-iam-attach-admin-policy-access-denied.png`](screenshots/cloudtrail-iam-attach-admin-policy-access-denied.png)
- [`cloudtrail-iam-create-access-key-access-denied.png`](screenshots/cloudtrail-iam-create-access-key-access-denied.png)
- [`cloudtrail-s3-put-bucket-policy-access-denied.png`](screenshots/cloudtrail-s3-put-bucket-policy-access-denied.png)

---

## Folder Guide

| Folder | Purpose |
|---|---|
| [`01-architecture/`](01-architecture/) | High-level architecture and access-control flow |
| [`02-setup-and-build/`](02-setup-and-build/) | Environment setup and build steps |
| [`03-operations-and-commands/`](03-operations-and-commands/) | AWS CLI commands used for testing and validation |
| [`04-defense/`](04-defense/) | Least-privilege design and hardening decisions |
| [`05-attacks-and-simulation/`](05-attacks-and-simulation/) | Controlled misuse testing and denied-action validation |
| [`06-reflections-and-improvements/`](06-reflections-and-improvements/) | Lessons learned and future improvements |
| [`policies/`](policies/) | IAM policy JSON files used in the lab |
| [`screenshots/`](screenshots/) | Screenshot evidence collected during the lab |

---

## Final Outcome

The final result was a restricted IAM user that could still perform its intended read-only S3 task, but could not perform unnecessary or higher-risk actions outside its role.

The lab validated that least privilege was successfully applied by proving:

- Required access still worked
- Broad S3 access was removed
- Unauthorized S3 writes were denied
- Destructive S3 actions were denied
- Bucket policy modification was denied
- IAM enumeration was denied
- Privilege escalation attempts were denied
- Credential persistence attempts were denied
- Denied activity was visible in CloudTrail

---

## Key Takeaway

Least privilege is not just about writing a smaller policy.

It is about proving that required access still works while unnecessary and risky access is removed.

This lab demonstrates that process with real AWS resources, AWS CLI testing, IAM policy remediation, denied-action validation, and CloudTrail audit evidence.
