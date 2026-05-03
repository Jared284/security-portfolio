# AWS IAM Least Privilege and Access Control Hardening

## Overview

This lab demonstrates how to identify and reduce over-permissioned AWS IAM access by testing allowed and denied actions, applying least-privilege policy design, and validating access activity through CloudTrail logs.

## Why I Built This Lab

IAM is one of the most important security layers in AWS. A single over-permissioned identity can create major cloud security risk if it can access resources, modify policies, or escalate privileges beyond its intended job function.

This lab is designed to show the full access-control hardening workflow: starting with an intentionally over-permissioned IAM identity, testing what that identity can do, replacing broad permissions with least privilege, and using CloudTrail to verify the results.

## Security Concepts Demonstrated

- Identity and Access Management
- Least privilege
- Access control
- IAM policy design
- Cloud misconfiguration risk
- Privilege escalation prevention
- CloudTrail audit logging
- Allowed and denied API activity validation

## AWS Services Used

- AWS IAM
- Amazon S3
- AWS CloudTrail
- AWS CLI

## Lab Workflow

1. Create a test IAM identity.
2. Attach an intentionally broad initial policy.
3. Test allowed and risky access using the AWS CLI.
4. Capture allowed and denied activity in CloudTrail.
5. Replace broad access with a scoped least-privilege policy.
6. Retest access after remediation.
7. Document final hardening recommendations.

## Folder Guide

| Folder | Purpose |
|---|---|
| `01-architecture/` | High-level architecture and access-control flow |
| `02-setup-and-build/` | Environment setup and build steps |
| `03-operations-and-commands/` | AWS CLI commands used for testing and validation |
| `04-defense/` | Least-privilege design and hardening decisions |
| `05-attacks-and-simulation/` | Controlled misuse testing and denied action validation |
| `06-reflections-and-improvements/` | Lessons learned and future improvements |
| `policies/` | IAM policy JSON files used in the lab |
| `screenshots/` | Screenshots and evidence collected during the lab |

## Final Outcome

By the end of this lab, the test IAM identity should only be able to perform the specific actions required for its intended role. Unnecessary access should be removed, risky actions should be denied, and CloudTrail should provide evidence of both allowed and denied API activity.
