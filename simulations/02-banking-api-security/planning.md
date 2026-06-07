# Planning

## Overview

This file documents the final planning scope for the AWS Banking API Security Lab.

The lab was designed to demonstrate a Broken Object Level Authorization (BOLA) vulnerability in a serverless AWS banking-style API, then remediate the issue using backend ownership validation.

The final project focused on one core endpoint:

~~~http
GET /accounts/{accountId}
~~~

The goal was to build, exploit, fix, and validate one realistic API authorization flaw instead of overexpanding the lab into a full banking platform.

---

## Final Lab Scope

The final lab scope included:

- Building a serverless API with Amazon API Gateway, AWS Lambda, and DynamoDB
- Creating simulated banking account records
- Implementing an intentionally vulnerable account lookup function
- Demonstrating unauthorized object access by modifying `accountId`
- Adding ownership validation in Lambda
- Re-testing authorized and unauthorized requests
- Documenting evidence with screenshots
- Identifying realistic production improvements

---

## Primary Security Scenario

The lab simulates a banking API where each user should only be able to access their own account.

Example ownership model:

| User | Owned Account |
|---|---|
| `user-001` | `acc-123` |
| `user-002` | `acc-456` |

The security rule is:

~~~text
A requester should only receive account data if they own the requested account.
~~~

The vulnerable version failed to enforce this rule.

---

## Primary Endpoint

The main endpoint tested in this lab was:

~~~http
GET /accounts/{accountId}
~~~

Example legitimate request:

~~~http
GET /accounts/acc-123
~~~

Example unauthorized request attempt:

~~~http
GET /accounts/acc-456
~~~

The `{accountId}` value is user-controlled, which makes it the key object identifier in the BOLA test.

---

## Data Model

The lab used a simplified banking data model.

### Account

| Field | Purpose |
|---|---|
| `accountId` | Unique account identifier |
| `ownerUserId` | User who owns the account |
| `accountType` | Simulated account type |
| `balance` | Simulated account balance |

Example account records:

| Account ID | Owner User ID | Account Type |
|---|---|---|
| `acc-123` | `user-001` | Checking |
| `acc-456` | `user-002` | Savings |

The `ownerUserId` field is the critical security field because it allows the backend to verify ownership.

---

## Vulnerability Details

The vulnerability tested was Broken Object Level Authorization.

The vulnerable API returned account data based only on the requested `accountId`.

Vulnerable logic:

~~~text
If accountId exists:
    return account data
~~~

This is insecure because the requester controls the object identifier.

The missing check was:

~~~text
Does the requester own this account?
~~~

Without that check, a requester could modify the path parameter and retrieve another user's account data.

---

## Attack Flow

The attack flow was:

1. Send a normal request for `acc-123`
2. Confirm account data is returned
3. Modify the request path to `acc-456`
4. Send the modified request
5. Observe that the vulnerable API returns another user's account data
6. Confirm that object-level authorization is missing

Attack pattern:

~~~text
/accounts/acc-123 → legitimate account access
/accounts/acc-456 → unauthorized object access
~~~

This confirmed the BOLA vulnerability.

---

## Remediation Plan

The remediation was to add object-level authorization in Lambda.

The secure Lambda function checks:

1. The requested `accountId`
2. The requester identity from `x-user-id`
3. The account owner stored in DynamoDB
4. Whether the requester identity matches the account owner

Secure rule:

~~~text
requesterUserId == account.ownerUserId
~~~

Secure logic:

~~~text
If account exists AND requester owns account:
    return account data
Else:
    return 403 Unauthorized
~~~

---

## Validation Plan

The fix was validated with two core tests.

| Test | Requester | Requested Account | Expected Result |
|---|---|---|---|
| Authorized access | `user-001` | `acc-123` | Allowed |
| Unauthorized access | `user-001` | `acc-456` | Denied |

The remediation was successful only if:

- legitimate access still worked
- unauthorized object access was blocked

---

## AWS Services Planned and Used

| Service | Purpose |
|---|---|
| Amazon API Gateway | Public API route |
| AWS Lambda | Backend account lookup and authorization logic |
| Amazon DynamoDB | Simulated account records |
| AWS IAM | Lambda permission to access DynamoDB |
| Amazon CloudWatch Logs | Lambda execution visibility and troubleshooting |

---

## Evidence Plan

The lab required screenshot evidence for each major phase.

| Evidence | Purpose |
|---|---|
| `api-gateway-route.png` | Prove API Gateway route existed |
| `api-gateway-request.png` | Show API request testing |
| `lambda-code-get-account.png` | Show vulnerable Lambda logic |
| `lambda-permission-error.png` | Show IAM troubleshooting |
| `lambda-test-acc-123.png` | Show Lambda retrieving account 123 |
| `lambda-test-acc-456.png` | Show Lambda retrieving account 456 |
| `transactions-records.png` | Show backend test records |
| `bola-request-acc-123.png` | Show legitimate account request |
| `bola-request-acc-456.png` | Show unauthorized object access |
| `lambda-authorization-check.png` | Show secure authorization logic |
| `authorized-request.png` | Show valid access after remediation |
| `blocked-request.png` | Show unauthorized access blocked |

---

## Final Folder Structure

The final lab structure was planned as:

~~~text
02-banking-api-security/
├── README.md
├── planning.md
├── 01-architecture/
├── 02-setup-and-build/
├── 03-operations-and-commands/
├── 04-defense/
├── 05-attacks-and-simulation/
├── 06-reflections-and-improvements/
└── screenshots/
~~~

This structure separates the project into architecture, build, operations, attack, defense, reflection, and evidence.

---

## Out-of-Scope Items

The lab intentionally did not implement every possible banking API feature.

The following items were considered future improvements, not part of the final core scope:

- Full login system
- Amazon Cognito authentication
- JWT validation
- API Gateway authorizers
- Full transaction endpoint implementation
- Transfer endpoint implementation
- WAF rate limiting
- SIEM integration
- Automated test suite
- Infrastructure-as-code deployment

These were excluded to keep the project focused on one complete and well-documented BOLA workflow.

---

## Production Improvements

A production-ready version should include:

- Amazon Cognito or another trusted identity provider
- JWT-based authentication
- API Gateway authorizers
- Object-level authorization on every account and transaction endpoint
- Structured Lambda logging
- CloudWatch metric filters for denied access attempts
- AWS WAF rate limiting
- Automated unit and integration tests
- Centralized logging into a SIEM
- Infrastructure-as-code using Terraform or CloudFormation

---

## Final Planning Decision

The final project intentionally prioritized depth over scope.

Instead of building many incomplete banking features, the lab focused on one strong security workflow:

~~~text
Build vulnerable API
        ↓
Exploit BOLA
        ↓
Remediate authorization logic
        ↓
Validate allowed and denied access
        ↓
Document evidence
~~~

This made the lab more useful as a cybersecurity portfolio project because it clearly demonstrates cloud security, API security, secure coding, attack simulation, and remediation validation.

---

## Key Takeaway

The planning goal was to prove a complete security concept, not to build a full banking application.

The final lab successfully demonstrates how a serverless API can expose sensitive data when it trusts user-controlled object identifiers, and how backend ownership validation prevents that failure.
