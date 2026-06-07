# Reflections and Improvements

## Overview

This section summarizes the key lessons learned from building, exploiting, and remediating the AWS Banking API Security Lab.

The lab demonstrated a Broken Object Level Authorization (BOLA) vulnerability in a serverless AWS API and showed how backend ownership validation can prevent unauthorized object access.

The main lesson is that an API can work correctly from a functionality perspective while still being insecure from an authorization perspective.

---

## Lab Summary

This lab followed a full security workflow:

~~~text
Build serverless banking API
        ↓
Create vulnerable account lookup logic
        ↓
Test normal account access
        ↓
Modify object identifier
        ↓
Confirm unauthorized account access
        ↓
Add ownership validation
        ↓
Retest allowed and denied access
        ↓
Document evidence and improvements
~~~

The project focused on the security risk created when an API trusts a user-controlled object identifier without verifying ownership.

---

## Core Vulnerability

The vulnerable endpoint was:

~~~http
GET /accounts/{accountId}
~~~

The vulnerable Lambda function returned account data based only on the supplied `accountId`.

The vulnerable logic was:

~~~text
If account exists:
    return account data
~~~

The secure logic should be:

~~~text
If account exists AND requester owns account:
    return account data
Else:
    deny access
~~~

The missing ownership check created the Broken Object Level Authorization vulnerability.

---

## Key Lesson 1: Functionality Is Not Security

The vulnerable API worked.

It accepted requests, queried DynamoDB, and returned account data.

The issue was that it returned data to the wrong requester.

That is an important security lesson:

~~~text
A system can be functional and still be insecure.
~~~

The API did what it was programmed to do, but the program logic failed to enforce authorization.

---

## Key Lesson 2: Object Identifiers Are Not Authorization

The vulnerable API treated knowledge of an `accountId` as enough to retrieve the account.

That is unsafe.

An object identifier only identifies a resource. It does not prove the requester is allowed to access that resource.

In this lab:

| Value | Meaning |
|---|---|
| `acc-123` | Account object identifier |
| `user-001` | Requester identity |
| `ownerUserId` | Stored account owner |
| `requesterUserId == ownerUserId` | Authorization decision |

The secure version required both the object identifier and the ownership relationship.

---

## Key Lesson 3: Authorization Must Happen Server-Side

The fix had to happen in backend logic.

A client should never be trusted to enforce access control.

The secure Lambda function enforced authorization by comparing:

~~~text
requesterUserId == account.ownerUserId
~~~

This prevented a requester from accessing another user's account by changing the URL path.

---

## Key Lesson 4: IAM and Application Authorization Are Different

During setup, the Lambda function needed IAM permissions to read from DynamoDB.

That was an AWS service permission issue.

However, fixing IAM permissions did not fix the BOLA vulnerability.

The distinction is important:

| Control | Question It Answers |
|---|---|
| IAM | Can Lambda access DynamoDB? |
| Application authorization | Should this requester receive this account record? |

Both controls matter, but they solve different problems.

---

## Key Lesson 5: Validation Must Include Negative Tests

It is not enough to test only the happy path.

The API needed to be tested with both:

- A valid access request
- An unauthorized access attempt

The important test was not just:

~~~text
Can user-001 access acc-123?
~~~

It was also:

~~~text
Can user-001 access acc-456?
~~~

Security testing requires proving that unauthorized behavior is blocked.

---

## Before-and-After Result

| Test | Vulnerable Version | Secure Version |
|---|---|---|
| `user-001` requests `acc-123` | Allowed | Allowed |
| `user-001` requests `acc-456` | Allowed | Denied |
| API returns account data | Yes | Yes, only when authorized |
| Ownership validation | No | Yes |
| BOLA risk | Present | Remediated |

The fix preserved legitimate functionality while blocking unauthorized object access.

---

## What Went Well

The lab successfully demonstrated:

- Serverless API construction using AWS
- DynamoDB-backed account lookup
- Lambda backend logic
- API Gateway route testing
- BOLA exploitation through object identifier modification
- IAM troubleshooting during Lambda setup
- Authorization remediation
- Before-and-after validation
- Screenshot-backed documentation

The strongest part of the lab is that the vulnerability was not just described. It was built, tested, exploited, fixed, and re-tested.

---

## Main Limitation

The biggest limitation is identity handling.

This lab uses:

~~~http
x-user-id: user-001
~~~

as a simulated requester identity.

That is acceptable for a controlled lab focused on authorization logic, but it is not production-grade authentication.

In a real system, a client-controlled header should not be trusted as the source of identity.

---

## Current Limitations

Current limitations include:

- `x-user-id` is simulated and client-controlled
- No Amazon Cognito authentication is implemented
- No JWT validation is implemented
- No API Gateway authorizer is used
- Only the account lookup endpoint is fully tested
- Transaction-level authorization is not fully implemented
- No automated unit or integration tests are included
- No CloudWatch metric filters are configured for denied access attempts
- No AWS WAF rate limiting is included
- No SIEM integration is included
- No production-grade logging pipeline is included

These limitations are acceptable for the scope of this lab, but they define clear future improvements.

---

## Production Improvement 1: Use Cognito or JWT-Based Authentication

A production version should replace the simulated `x-user-id` header with a trusted identity source.

Better options include:

- Amazon Cognito
- OIDC identity provider
- SAML federation
- JWT-based authentication
- API Gateway JWT authorizer

The Lambda function should receive verified identity claims, not trust a raw client-controlled header.

---

## Production Improvement 2: Add API Gateway Authorizers

API Gateway could be improved with an authorizer layer.

Possible authorizer options:

- Cognito user pool authorizer
- JWT authorizer
- Lambda authorizer

This would help verify who the requester is before the request reaches the backend Lambda function.

However, an authorizer would not replace object-level authorization. Lambda would still need to verify whether the requester owns the requested account.

---

## Production Improvement 3: Add Transaction-Level Authorization

This lab focused mainly on account lookup.

A more complete banking API would also enforce authorization on transaction endpoints.

Example endpoint:

~~~http
GET /accounts/{accountId}/transactions
~~~

This endpoint should use the same rule:

~~~text
Only return transactions if the requester owns the account.
~~~

Without transaction-level authorization, transaction history could still be exposed even if account lookup is protected.

---

## Production Improvement 4: Add Security Logging

The secure Lambda function should log authorization decisions in structured JSON.

Useful events to log:

- Account lookup attempt
- Requested account ID
- Requester identity
- Authorization allowed
- Authorization denied
- Denial reason
- Owner mismatch
- Missing identity
- Repeated denied access attempts

Example log structure:

~~~json
{
  "event_type": "authorization_denied",
  "reason": "owner_mismatch",
  "requesterUserId": "user-001",
  "accountId": "acc-456",
  "ownerUserId": "user-002"
}
~~~

Structured logs would make detection and alerting easier.

---

## Production Improvement 5: Add Detection and Alerting

Future versions could add CloudWatch-based detection for suspicious authorization behavior.

Useful detections:

- Multiple `403` responses from the same requester
- Repeated access attempts against different account IDs
- Missing identity headers
- Sequential account ID probing
- Repeated owner mismatch events

Possible AWS services:

- CloudWatch Logs
- CloudWatch Metric Filters
- CloudWatch Alarms
- SNS notifications
- EventBridge rules

This would turn the lab from application security remediation into detection engineering as well.

---

## Production Improvement 6: Add Rate Limiting and WAF

A production API should include controls to reduce automated abuse.

Possible protections:

- API Gateway throttling
- AWS WAF rate-based rules
- IP-based blocking
- Request validation
- Bot protection patterns

These controls would not fix BOLA by themselves, but they would reduce the scale and speed of exploitation attempts.

---

## Production Improvement 7: Add Automated Tests

The authorization logic should be covered by automated tests.

Important test cases:

| Test Case | Expected Result |
|---|---|
| Owner requests own account | Allowed |
| User requests another user's account | Denied |
| Missing identity header | Denied |
| Nonexistent account | Not found |
| Invalid account ID format | Bad request |
| Transaction request for owned account | Allowed |
| Transaction request for another user's account | Denied |

Automated tests would help prevent future code changes from reintroducing the vulnerability.

---

## Production Improvement 8: Improve Error Handling

The secure implementation should return clear and consistent responses.

Possible response design:

| Condition | Response |
|---|---|
| Missing `accountId` | `400 Bad Request` |
| Missing requester identity | `401 Unauthorized` |
| Account does not exist | `404 Not Found` |
| Requester does not own account | `403 Forbidden` |
| Requester owns account | `200 OK` |

This would make the API easier to debug and safer to operate.

---

## Production Improvement 9: Centralize Logs in a SIEM

For a more advanced version, API and Lambda logs could be forwarded into a SIEM.

Possible SIEM destinations:

- Splunk
- Elastic
- Microsoft Sentinel
- Datadog
- AWS Security Lake

This would allow deeper investigation of suspicious account access patterns.

---

## Skills Demonstrated

This lab demonstrates practical skills across multiple cybersecurity areas:

| Skill Area | Evidence |
|---|---|
| Cloud security | AWS API Gateway, Lambda, DynamoDB |
| Application security | BOLA testing and remediation |
| IAM troubleshooting | Lambda-to-DynamoDB permission issue |
| Secure coding | Ownership validation logic |
| API testing | Before-and-after account access requests |
| Threat modeling | Trust boundaries and attack flow |
| Detection thinking | Future logging and alerting improvements |
| Documentation | Evidence-backed GitHub write-up |

---

## Security Takeaway

The main security lesson is:

~~~text
Authentication identifies the requester.
Authorization decides what the requester can access.
~~~

Even if a user is authenticated, they should not automatically be able to access every object in the system.

For sensitive APIs, authorization must be enforced at the object level.

---

## Final Reflection

This lab showed how a small backend logic mistake can create a serious security issue.

The vulnerable API did not fail because AWS was misconfigured or because DynamoDB returned the wrong data. It failed because the application did not verify whether the requester owned the account being requested.

The remediation was simple but important: compare the requester identity to the account owner before returning data.

That single authorization check changed the API from vulnerable object retrieval to secure object access control.

---

## Key Takeaway

Broken Object Level Authorization is dangerous because it can hide inside APIs that appear to work normally.

The fix is not just to build working endpoints. The fix is to enforce authorization every time a requester asks for sensitive object-level data.
