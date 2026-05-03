# Architecture

## Purpose

This section documents the high-level architecture for the AWS IAM least-privilege hardening lab.

The lab is built around a test IAM identity that initially has broader permissions than required. Access is tested through the AWS CLI, logged through CloudTrail, and then remediated with a scoped least-privilege policy.

## High-Level Flow

```text
+-------------------------------+
| IAM User: lab4-junior-analyst |
+---------------+---------------+
                |
                v
+-------------------------------+
| Initial IAM Policy            |
| Overly broad permissions      |
+---------------+---------------+
                |
                v
+-------------------------------+
| AWS Resource Access Testing   |
| S3 + IAM CLI commands         |
+---------------+---------------+
                |
                v
+-------------------------------+
| CloudTrail                    |
| Allowed and denied API events |
+---------------+---------------+
                |
                v
+-------------------------------+
| Least-Privilege Remediation   |
| Scoped custom IAM policy      |
+---------------+---------------+
                |
                v
+-------------------------------+
| Final Validation              |
| Allowed needed access         |
| Denied excessive access       |
+-------------------------------+
