# Lessons Learned

## Overview

This file summarizes the main lessons learned from building, exploiting, and remediating the AWS Banking API Security Lab.

The lab focused on a Broken Object Level Authorization (BOLA) vulnerability in a serverless banking-style API built with API Gateway, Lambda, and DynamoDB.

The most important lesson is that a working API is not automatically a secure API.

---

## Lesson 1: Functionality Does Not Prove Security

The vulnerable API worked from a basic functionality perspective.

It could:

- Receive API requests
- Extract an `accountId`
- Query DynamoDB
- Return account data

However, it did not verify whether the requester was allowed to access the requested account.

That means the API was functional but insecure.

Key takeaway:

~~~text
An API can return the correct data and still expose that data to the wrong user.
~~~

---

## Lesson 2: Object Identifiers Are Not Authorization

The vulnerable API treated the `accountId` path parameter as enough information to return account data.

Example:

~~~http
GET /accounts/acc-456
~~~

The problem is that `acc-456` only identifies an object. It does not prove that the requester is allowed to access that object.

A secure API must check both:

~~~text
What object is being requested?
Who is requesting it?
~~~

In this lab, the secure authorization relationship was:

~~~text
requesterUserId == account.ownerUserId
~~~

Without that comparison, the API was vulnerable to BOLA.

---

## Lesson 3: BOLA Is a Backend Logic Failure

The BOLA vulnerability was not caused by API Gateway, DynamoDB, or AWS automatically doing something wrong.

The issue existed because the Lambda function returned account data without enforcing ownership validation.

Vulnerable backend logic:

~~~text
If account exists:
    return account data
~~~

Secure backend logic:

~~~text
If account exists AND requester owns account:
    return account data
Else:
    deny access
~~~

The fix had to happen where the application understood the relationship between the requester and the account.

---

## Lesson 4: Authentication and Authorization Are Different

This lab used `x-user-id` to simulate requester identity.

Even if a user is identified, that does not automatically mean they should be able to access every object in the system.

Authentication answers:

~~~text
Who is the requester?
~~~

Authorization answers:

~~~text
What is the requester allowed to access?
~~~

The vulnerable API failed because it did not enforce authorization at the object level.

---

## Lesson 5: IAM Permissions and Application Authorization Are Different

During setup, the Lambda function initially had a DynamoDB permission issue.

That was an IAM problem.

However, fixing IAM permissions did not fix the BOLA vulnerability.

The difference matters:

| Control | Question It Answers |
|---|---|
| IAM | Can Lambda access DynamoDB? |
| Application authorization | Should this requester receive this account record? |

IAM allowed the AWS service to access the database.

Application authorization controlled whether the user should receive the returned data.

Both are necessary, but they solve different security problems.

---

## Lesson 6: Negative Testing Is Required

Testing only the expected path is not enough.

The API needed to be tested with both:

- A valid request
- An unauthorized request

Valid request:

~~~text
user-001 requests acc-123
Expected result: allowed
~~~

Unauthorized request:

~~~text
user-001 requests acc-456
Expected result: denied
~~~

The vulnerable version failed because the unauthorized request succeeded.

Security testing requires proving that bad actions are blocked, not just that good actions work.

---

## Lesson 7: Backend Ownership Data Is Critical

The data model needed an ownership field to support authorization.

In this lab, that field was:

~~~text
ownerUserId
~~~

Without `ownerUserId`, the backend would not have a reliable way to determine who owns an account.

A secure object-level authorization system needs:

~~~text
Object identifier
+
Object owner
+
Requester identity
+
Authorization comparison
~~~

The vulnerable version used only the object identifier.

The secure version used the ownership relationship.

---

## Lesson 8: Serverless APIs Still Need Explicit Security Logic

Using serverless services does not remove the need for secure application logic.

API Gateway routed the request.

Lambda processed the request.

DynamoDB returned the account record.

But none of those services automatically decided whether the requester should be allowed to access that specific account.

That decision had to be implemented in the Lambda function.

Key takeaway:

~~~text
Serverless architecture reduces infrastructure management, not application security responsibility.
~~~

---

## Lesson 9: A Simple Bug Can Have Serious Impact

The vulnerable logic was simple, but the impact could be severe in a real banking or fintech system.

Potential real-world impact:

- Unauthorized account data exposure
- Customer privacy violations
- Broken customer isolation
- Exposure of balances or transaction history
- Compliance risk
- Fraud risk if write actions are also vulnerable
- Loss of customer trust

BOLA vulnerabilities are dangerous because they can hide inside APIs that appear normal.

---

## Lesson 10: Remediation Must Preserve Legitimate Access

A good security fix should not simply break the API.

The secure version needed to prove two things:

1. Valid users could still access their own accounts.
2. Invalid access attempts were blocked.

Validation result:

| Test | Expected Result |
|---|---|
| `user-001` requests `acc-123` | Allowed |
| `user-001` requests `acc-456` | Denied |

This proved that the remediation worked without destroying normal functionality.

---

## Lesson 11: Error Responses Matter

Clear error handling improves both security and debugging.

Useful response behavior:

| Condition | Response |
|---|---|
| Missing account ID | `400 Bad Request` |
| Missing requester identity | `401 Unauthorized` |
| Account does not exist | `404 Not Found` |
| Requester does not own account | `403 Unauthorized` |
| Requester owns account | `200 OK` |

The most important authorization response in this lab was:

~~~text
403 Unauthorized
~~~

That response showed that the backend recognized the requester but denied access to the requested object.

---

## Lesson 12: Security Logging Would Make the Lab Stronger

The lab proved exploitation and remediation, but a stronger production version would also log authorization decisions.

Useful security events:

- Account lookup attempted
- Requested `accountId`
- Requester identity
- Authorization allowed
- Authorization denied
- Owner mismatch
- Missing identity
- Repeated denied requests

Example structured log:

~~~json
{
  "event_type": "authorization_denied",
  "reason": "owner_mismatch",
  "requesterUserId": "user-001",
  "accountId": "acc-456",
  "ownerUserId": "user-002"
}
~~~

These logs could support CloudWatch metric filters, alarms, and SIEM detection rules.

---

## Lesson 13: Production Identity Should Not Trust Client-Controlled Headers

The lab used this header for simplicity:

~~~http
x-user-id: user-001
~~~

That is acceptable for testing authorization logic in a controlled lab.

It is not acceptable as production authentication.

In production, identity should come from a trusted source such as:

- Amazon Cognito
- OIDC provider
- SAML federation
- JWT claims
- API Gateway authorizer

The backend should validate trusted identity claims before making authorization decisions.

---

## Lesson 14: API Gateway Authorizers Help, But They Do Not Replace Object-Level Authorization

A production API should use an authorizer to verify identity before the request reaches Lambda.

However, an authorizer alone does not solve BOLA.

An authorizer can confirm:

~~~text
This requester is authenticated.
~~~

But Lambda still needs to confirm:

~~~text
This requester owns the specific account being requested.
~~~

Both controls are needed.

---

## Lesson 15: Detection Can Build on Authorization Logic

Once the API logs authorization decisions, the same lab could become a detection engineering project.

Possible detections:

- Multiple `403` responses from the same requester
- One requester attempting many account IDs
- Missing identity headers
- Sequential account ID probing
- Repeated owner mismatch events

Potential AWS detection stack:

- CloudWatch Logs
- CloudWatch Metric Filters
- CloudWatch Alarms
- SNS notifications
- EventBridge rules
- SIEM forwarding

This would extend the lab from AppSec remediation into cloud detection engineering.

---

## What Went Well

The lab successfully showed the full security workflow:

- Built a serverless API
- Created a vulnerable account lookup endpoint
- Stored test records in DynamoDB
- Tested Lambda and API Gateway behavior
- Demonstrated unauthorized object access
- Added ownership validation
- Re-tested allowed and denied access
- Documented the vulnerability and remediation with screenshots

The strongest part of the lab is that the vulnerability was not theoretical. It was built, exploited, fixed, and validated.

---

## What Could Be Improved

Future improvements include:

- Replace `x-user-id` with Cognito or JWT claims
- Add API Gateway authorizers
- Add transaction-level authorization
- Add structured security logging
- Add CloudWatch metric filters for denied access attempts
- Add AWS WAF rate limiting
- Add automated authorization tests
- Add SIEM forwarding
- Add infrastructure-as-code using Terraform or CloudFormation
- Add a cleaner architecture diagram

These improvements would make the lab more production-like and more advanced.

---

## Final Takeaway

The main lesson from this lab is:

~~~text
Never trust a user-controlled object identifier as proof of authorization.
~~~

The vulnerable API returned account data because the requested `accountId` existed.

The secure API returned account data only when the requester owned the account.

That difference is the core of preventing Broken Object Level Authorization.
