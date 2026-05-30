# Lambda Function: getAccount Secure Implementation

## Overview

This file documents the secure version of the `getAccount` Lambda function used in the AWS Banking API Security Lab.

The vulnerable version returned account data based only on the `accountId` supplied in the request path. The secure version adds object-level authorization by verifying that the requester owns the requested account before returning data.

The goal of this implementation was to fix the Broken Object Level Authorization (BOLA) vulnerability while preserving legitimate account access.

---

## Purpose of the Secure Function

The secure `getAccount` function acts as the backend authorization enforcement point for the account lookup API.

It is connected to this route:

~~~http
GET /accounts/{accountId}
~~~

The function now makes an authorization decision before returning account data.

It checks:

1. What account is being requested?
2. Who is making the request?
3. Does the requester own the requested account?

Only if ownership matches does the function return the account record.

---

## Secure Access Logic

The secure implementation uses both the requested object and the requester identity.

Secure decision flow:

~~~text
Request contains accountId
        +
Request contains requester identity
        ↓
Lambda queries DynamoDB
        ↓
Lambda reads account owner
        ↓
Lambda compares requester to owner
        ↓
Allow if match
Deny if mismatch
~~~

The key authorization rule is:

~~~text
requesterUserId == account.ownerUserId
~~~

---

## Request Inputs

The secure Lambda function uses two important request values.

### 1. Account ID

The `accountId` comes from the API path.

Example request:

~~~http
GET /accounts/acc-123
~~~

Extracted value:

~~~text
accountId = acc-123
~~~

### 2. Requester Identity

The requester identity is simulated with the `x-user-id` header.

Example header:

~~~http
x-user-id: user-001
~~~

Extracted value:

~~~text
requesterUserId = user-001
~~~

Important limitation:

~~~text
x-user-id is used only for lab testing.
~~~

In production, requester identity should come from a trusted authentication provider such as Amazon Cognito, OIDC, SAML, or verified JWT claims.

---

## Secure Function Behavior

The secure Lambda function performs the following steps:

1. Extract `accountId` from the request path.
2. Extract `x-user-id` from the request headers.
3. Query DynamoDB for the requested account.
4. Return `404 Not Found` if the account does not exist.
5. Read the account's `ownerUserId`.
6. Compare `ownerUserId` to the requester identity.
7. Return `403 Unauthorized` if ownership does not match.
8. Return account data if ownership matches.

---

## Secure Pseudocode

~~~text
accountId = request.pathParameters.accountId
requesterUserId = request.headers.x-user-id

account = DynamoDB.getItem(accountId)

if account does not exist:
    return 404 Not Found

if account.ownerUserId != requesterUserId:
    return 403 Unauthorized

return account
~~~

This logic prevents a requester from accessing another user's account by changing the account ID in the URL.

---

## Before-and-After Comparison

### Vulnerable Version

~~~text
If account exists:
    return account data
~~~

Problem:

~~~text
The function does not check who owns the account.
~~~

---

### Secure Version

~~~text
If account exists AND requester owns account:
    return account data

If account exists BUT requester does not own account:
    return 403 Unauthorized
~~~

Improvement:

~~~text
The function enforces object-level authorization before returning sensitive data.
~~~

---

## Authorized Request Example

Test case:

~~~http
GET /accounts/acc-123
x-user-id: user-001
~~~

Account ownership:

~~~text
acc-123 ownerUserId = user-001
requesterUserId = user-001
~~~

Expected result:

~~~text
200 OK
Account data returned
~~~

Security meaning:

~~~text
The requester owns the account, so access is allowed.
~~~

Evidence:

- `authorized-request.png`

---

## Unauthorized Request Example

Test case:

~~~http
GET /accounts/acc-456
x-user-id: user-001
~~~

Account ownership:

~~~text
acc-456 ownerUserId = user-002
requesterUserId = user-001
~~~

Expected result:

~~~text
403 Unauthorized
Access denied
~~~

Security meaning:

~~~text
The requester does not own the account, so access is denied.
~~~

Evidence:

- `blocked-request.png`

---

## Validation Results

| Requester | Requested Account | Account Owner | Expected Result | Security Meaning |
|---|---|---|---|---|
| `user-001` | `acc-123` | `user-001` | Allowed | Legitimate access preserved |
| `user-001` | `acc-456` | `user-002` | Denied | Unauthorized object access blocked |

The secure implementation is successful because it does both:

- Allows valid account access
- Blocks unauthorized account access

---

## Evidence

Evidence related to the secure Lambda implementation includes:

| Screenshot | What It Shows |
|---|---|
| `lambda-authorization-check.png` | Secure Lambda logic with ownership validation |
| `authorized-request.png` | Valid requester accessing their own account |
| `blocked-request.png` | Unauthorized requester blocked from another account |
| `bola-request-acc-456.png` | Original vulnerable behavior before remediation |
| `lambda-test-acc-456.png` | Backend could retrieve the object before authorization was added |

---

## Security Impact

The secure Lambda function fixes the main BOLA issue by enforcing ownership before returning account data.

Before remediation:

~~~text
Any valid accountId could return account data.
~~~

After remediation:

~~~text
Only the account owner can receive account data.
~~~

This prevents unauthorized access caused by changing object identifiers in the API path.

---

## Why This Fix Belongs in Lambda

API Gateway routes the request, but Lambda understands the application data and authorization relationship.

Lambda has access to:

- The requested `accountId`
- The requester identity
- The DynamoDB account record
- The account's `ownerUserId`

Because Lambda can compare these values, it is the correct place in this lab to enforce object-level authorization.

---

## Error Handling

The secure function should return different responses depending on the request state.

| Condition | Response | Meaning |
|---|---|---|
| Missing account ID | `400 Bad Request` | Request is malformed |
| Missing requester identity | `401 Unauthorized` | Requester identity is not present |
| Account does not exist | `404 Not Found` | Requested object does not exist |
| Requester does not own account | `403 Unauthorized` | Requester is not allowed to access object |
| Requester owns account | `200 OK` | Access allowed |

This creates clearer security behavior than returning account data for every valid object identifier.

---

## Security Logging

A stronger version of this function should log authorization decisions.

Useful log events include:

- Account lookup attempt
- Requester identity
- Requested account ID
- Authorization allowed
- Authorization denied
- Denial reason
- Owner mismatch events

Example structured log concept:

~~~json
{
  "event_type": "authorization_denied",
  "reason": "owner_mismatch",
  "requesterUserId": "user-001",
  "accountId": "acc-456",
  "ownerUserId": "user-002"
}
~~~

These logs could later support detection engineering through CloudWatch metric filters or SIEM alerts.

---

## Remaining Limitations

This secure implementation fixes the object-level authorization flaw, but it is still a lab implementation.

Current limitations:

- `x-user-id` is simulated and client-controlled
- No Cognito or JWT authorizer is implemented
- No signed token validation is performed
- No transaction-level authorization is fully implemented
- No automated tests are included
- No CloudWatch alerting for repeated denied requests is included
- No centralized SIEM pipeline is connected

The key security logic is correct for the lab, but production identity handling would need to be stronger.

---

## Production Improvements

A production-ready version should include:

- Amazon Cognito authentication
- JWT claim validation
- API Gateway authorizers
- Authorization checks for all sensitive endpoints
- Structured JSON security logging
- CloudWatch metric filters for repeated denied access attempts
- AWS WAF rate limiting
- Automated unit and integration tests
- Centralized logging into a SIEM
- Alerting for suspicious object access patterns

---

## Key Takeaway

The secure Lambda function changes the API from simple account lookup to authorized account lookup.

The vulnerable function only checked whether the requested account existed. The secure function checks whether the requester owns the requested account before returning data.

That ownership check is what remediates the Broken Object Level Authorization vulnerability.
