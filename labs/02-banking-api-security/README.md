# AWS Banking API Security Lab: BOLA Attack and Authorization Hardening

## Overview

This lab demonstrates a Broken Object Level Authorization (BOLA) vulnerability in a serverless AWS banking API.

I built a cloud-based API using Amazon API Gateway, AWS Lambda, and Amazon DynamoDB. The first version intentionally trusted a user-controlled `accountId` path parameter and returned account data without verifying ownership. I then exploited that insecure behavior through controlled API requests, remediated the issue by adding backend ownership validation in Lambda, and validated that legitimate access still worked while unauthorized access was blocked.

The goal was to document the full application security workflow:

~~~text
Serverless API Build
        ↓
Vulnerable Object Access
        ↓
Controlled BOLA Exploitation
        ↓
Unauthorized Account Data Exposure
        ↓
Authorization Logic Remediation
        ↓
Allowed / Denied Request Validation
        ↓
Evidence-Based Documentation
~~~

This lab is intentionally focused on API authorization logic in a banking-style environment.

---

## What This Lab Proves

This project demonstrates the ability to:

- Build a serverless API using API Gateway, Lambda, and DynamoDB
- Model account ownership in a banking-style data system
- Identify Broken Object Level Authorization risk
- Exploit object-level authorization failure through controlled API requests
- Validate unauthorized access to another user's account data
- Remediate BOLA using backend ownership validation
- Preserve legitimate access after remediation
- Block unauthorized access with a `403` response
- Document vulnerable behavior, secure behavior, and evidence
- Explain why functional APIs can still be insecure without explicit authorization checks

---

## AWS Services Used

- Amazon API Gateway
- AWS Lambda
- Amazon DynamoDB
- Amazon CloudWatch Logs
- AWS IAM

---

## Architecture Summary

~~~text
Client / Tester
        ↓
API Gateway
GET /accounts/{accountId}
        ↓
AWS Lambda: getAccount
        ↓
DynamoDB: Accounts Table
        ↓
Account Data Returned
~~~

The API exposes an account lookup endpoint:

~~~http
GET /accounts/{accountId}
~~~

The backend Lambda function retrieves account records from DynamoDB based on the requested `accountId`.

Architecture documentation:

- [Architecture](./01-architecture/README.md)

---

## Vulnerable Design

In the vulnerable implementation, the API trusted the `accountId` supplied in the request path.

~~~text
User-controlled accountId
        ↓
Lambda queries DynamoDB
        ↓
Account record returned
        ↓
No ownership validation
~~~

This created a direct object access vulnerability. A requester could change the object identifier from one account to another and receive account data that did not belong to them.

Example attack pattern:

~~~text
/accounts/acc-123   → legitimate account access
/accounts/acc-456   → unauthorized account access
~~~

The issue was not that the API failed to function. The issue was that it functioned without enforcing object-level authorization.

Vulnerable implementation documentation:

- [Vulnerable Lambda Implementation](./02-setup-and-build/lambda-get-account-vulnerable.md)
- [BOLA Attack Simulation](./05-attacks-and-simulation/bola-attack.md)

---

## Attack Summary

The controlled attack demonstrated that the API returned account data based only on the path parameter.

Attack steps:

1. Send a request for one account.
2. Modify the `accountId` in the request path.
3. Send the modified request.
4. Observe that the API returns a different user's account data.

The attack did not require malware, privilege escalation, or credential theft. It worked because the backend trusted a user-controlled object identifier without validating ownership.

Attack documentation:

- [BOLA Attack Simulation](./05-attacks-and-simulation/bola-attack.md)

---

## Remediation Summary

The Lambda function was updated to enforce object-level authorization before returning account data.

The secure implementation performs the following checks:

1. Extract `accountId` from the request path.
2. Extract requester identity from the `x-user-id` request header.
3. Query DynamoDB for the requested account.
4. Compare the requester identity to the stored account owner.
5. Return account data only if ownership matches.
6. Return `403 Unauthorized` if ownership does not match.

Remediated access flow:

~~~text
Requested accountId
        +
Requester Identity
        ↓
DynamoDB Account Lookup
        ↓
Ownership Validation
        ↓
Allow if owner matches / Deny if owner does not match
~~~

Defense documentation:

- [Authorization Fix](./04-defense/authorization-fix.md)
- [Secure Lambda Implementation](./04-defense/lambda-get-account-secure.md)

---

## Validation Results

| Test | Vulnerable Result | Secure Result | Security Meaning |
|---|---|---|---|
| User requests their own account | Allowed | Allowed | Required access preserved |
| User modifies account ID to another user's account | Allowed | Denied | BOLA remediated |
| Lambda retrieves account from DynamoDB | Works | Works | Backend functionality preserved |
| Unauthorized request | Account data exposed | `403 Unauthorized` | Object-level authorization enforced |

The most important result is that remediation did not simply break the API. Legitimate access still worked, while unauthorized object access was denied.

---

## Evidence Summary

Evidence collected during the lab includes:

- API Gateway route configuration
- API Gateway account request testing
- Lambda vulnerable implementation
- Lambda permission troubleshooting
- Lambda test events for multiple accounts
- DynamoDB account / transaction records
- Successful vulnerable BOLA request
- Updated Lambda authorization logic
- Successful authorized request after remediation
- Blocked unauthorized request after remediation

Screenshots are stored in the lab's `screenshots/` directory and referenced throughout the supporting documentation.

---

## Repository Structure

~~~text
labs/02-banking-api-security/
│
├── 01-architecture/
│   └── System design, trust boundaries, and vulnerable data flow
│
├── 02-setup-and-build/
│   └── DynamoDB setup, Lambda implementation, and API Gateway configuration
│
├── 03-operations-and-commands/
│   └── API request formats and testing commands
│
├── 04-defense/
│   └── Authorization remediation and secure Lambda implementation
│
├── 05-attacks-and-simulation/
│   └── Controlled BOLA exploitation and evidence
│
├── 06-reflections-and-improvements/
│   └── Lessons learned, limitations, and production improvements
│
└── screenshots/
    └── Evidence screenshots for build, attack, and remediation phases
~~~

---

## Security Concepts Demonstrated

- Broken Object Level Authorization (BOLA)
- Insecure Direct Object Reference-style access risk
- Server-side authorization enforcement
- API security testing
- Serverless application security
- AWS Lambda backend authorization logic
- DynamoDB object ownership modeling
- API Gateway routing
- CloudWatch log review
- Least privilege and access control design
- Secure remediation validation

---

## Why This Lab Matters

Banking and fintech systems depend on strict authorization boundaries. A user should never be able to access another customer's financial data by changing an identifier in an API request.

This lab demonstrates a common but severe API security failure: the application worked as designed, but the design did not enforce ownership.

The key lesson is that authentication alone is not enough. APIs must enforce authorization at the object level for every sensitive resource request.

---

## Limitations

This lab is intentionally scoped to focus on object-level authorization logic.

Current limitations:

- `x-user-id` is used to simulate authenticated identity
- No Amazon Cognito or JWT authorizer is implemented yet
- Only the account lookup endpoint is fully tested
- Transaction-level authorization is not fully implemented
- No WAF rate limiting is included in the final documented flow
- No automated unit tests are included
- No SIEM integration is included

In a production system, identity should come from a trusted authentication provider such as Amazon Cognito or another JWT-based identity provider. A client-controlled header should not be trusted as the source of identity in a real application.

---

## Future Improvements

Future improvements could include:

- Add Amazon Cognito authentication
- Use JWT claims instead of a simulated `x-user-id` header
- Add API Gateway authorizers
- Extend authorization checks to transaction endpoints
- Add CloudWatch metric filters for unauthorized access attempts
- Add AWS WAF rate limiting for repeated suspicious requests
- Add structured JSON logging in Lambda
- Add automated tests for authorized and unauthorized access cases
- Forward logs to a SIEM such as Splunk, Elastic, or Microsoft Sentinel

---

## Key Takeaway

A working API is not automatically a secure API.

The vulnerable version returned the requested data correctly, but it failed to verify whether the requester was allowed to access that data. The remediated version preserved legitimate functionality while enforcing object-level authorization and blocking unauthorized access.
