# Defense

## Overview

This section documents how the Broken Object Level Authorization (BOLA) vulnerability was remediated in the AWS Banking API Security Lab.

The vulnerable version of the API returned account data based only on the `accountId` supplied in the request path. The secure version adds backend ownership validation before returning account data.

The purpose of the defense phase was to prove that the API could still support legitimate account access while blocking unauthorized object access.

---

## Vulnerability Being Fixed

The vulnerable API trusted a user-controlled object identifier:

~~~http
GET /accounts/{accountId}
~~~

In the vulnerable version, a requester could change the account ID in the URL and retrieve another user's account data.

Example vulnerable pattern:

~~~text
Requester asks for acc-123
        ↓
API returns acc-123

Requester changes path to acc-456
        ↓
API returns acc-456
        ↓
Unauthorized object access succeeds
~~~

The root problem was missing object-level authorization.

---

## Remediation Strategy

The remediation added an ownership check inside the Lambda function.

Instead of returning account data based only on `accountId`, the function now checks whether the requester owns the requested account.

Secure decision logic:

~~~text
Requested accountId
        +
Requester identity
        ↓
Look up account in DynamoDB
        ↓
Compare requester identity to account owner
        ↓
Allow if match
Deny if mismatch
~~~

The important authorization rule is:

~~~text
requesterUserId == account.ownerUserId
~~~

---

## Secure Access Flow

The secure version follows this process:

1. Receive request for `GET /accounts/{accountId}`
2. Extract the requested `accountId`
3. Extract requester identity from the `x-user-id` header
4. Query DynamoDB for the requested account
5. Read the account's `ownerUserId`
6. Compare `x-user-id` to `ownerUserId`
7. Return the account only if ownership matches
8. Return `403 Unauthorized` if ownership does not match

Secure flow:

~~~text
Client request
        ↓
API Gateway
        ↓
Lambda extracts accountId
        ↓
Lambda extracts x-user-id
        ↓
Lambda queries DynamoDB
        ↓
Lambda validates ownership
        ↓
Allowed or denied response
~~~

---

## Authorization Logic

The vulnerable logic was:

~~~text
If account exists:
    return account data
~~~

The secure logic is:

~~~text
If account exists AND requester owns account:
    return account data
Else:
    return 403 Unauthorized
~~~

This changes the API from object retrieval to authorized object retrieval.

---

## Identity Handling

This lab uses the following request header to simulate requester identity:

~~~http
x-user-id: user-001
~~~

This allows the authorization logic to compare the requester to the account owner stored in DynamoDB.

Important limitation:

~~~text
x-user-id is a lab simplification, not production authentication.
~~~

In production, requester identity should come from a trusted identity provider such as Amazon Cognito, OIDC, SAML, or verified JWT claims.

---

## Remediation Test Cases

The same account access pattern was tested after the fix.

| Requester | Requested Account | Account Owner | Expected Result |
|---|---|---|---|
| `user-001` | `acc-123` | `user-001` | Allowed |
| `user-001` | `acc-456` | `user-002` | Denied |

The fix is successful only if both conditions are true:

- Authorized access still works
- Unauthorized access is blocked

---

## Authorized Request Validation

Test case:

~~~http
GET /accounts/acc-123
x-user-id: user-001
~~~

Expected result:

~~~text
200 OK
Account data returned
~~~

Security meaning:

~~~text
user-001 owns acc-123, so access is allowed.
~~~

Evidence:

- `authorized-request.png`

---

## Unauthorized Request Validation

Test case:

~~~http
GET /accounts/acc-456
x-user-id: user-001
~~~

Expected result:

~~~text
403 Unauthorized
Access denied
~~~

Security meaning:

~~~text
user-001 does not own acc-456, so access is denied.
~~~

Evidence:

- `blocked-request.png`

---

## Before-and-After Behavior

| Test | Vulnerable Version | Secure Version |
|---|---|---|
| Request own account | Allowed | Allowed |
| Request another user's account | Allowed | Denied |
| Query DynamoDB | Works | Works |
| Validate ownership | No | Yes |
| Protect account boundary | No | Yes |

The secure version preserved functionality while adding the missing authorization control.

---

## Evidence

Evidence collected during the defense phase includes:

| Screenshot | What It Shows |
|---|---|
| `lambda-authorization-check.png` | Updated Lambda logic with ownership validation |
| `authorized-request.png` | Valid user accessing their own account |
| `blocked-request.png` | Unauthorized access attempt blocked |
| `bola-request-acc-456.png` | Original vulnerable access before remediation |
| `lambda-test-acc-456.png` | Lambda could retrieve the object before access control was added |

---

## Security Impact

The remediation reduced the risk of unauthorized account data exposure.

Before the fix:

~~~text
Knowledge of a valid accountId was enough to retrieve account data.
~~~

After the fix:

~~~text
The requester must be authorized to access the requested account.
~~~

This prevents a requester from accessing another user's financial data by modifying the account ID in the URL.

---

## Why the Fix Works

The fix works because it moves the authorization decision into backend logic.

The backend now evaluates three things:

1. What account is being requested?
2. Who is making the request?
3. Does the requester own the account?

Only when the ownership relationship matches does the API return data.

This is the core requirement for preventing BOLA.

---

## Why This Matters

BOLA vulnerabilities are dangerous because they often exist in APIs that appear to work correctly.

The vulnerable API did return the requested account record. The problem was that it did not check whether the requester should receive that record.

In banking, fintech, SaaS, healthcare, and cloud platforms, this type of flaw can expose sensitive data across users, tenants, or customers.

---

## Production Improvements

This lab focused on backend object-level authorization.

A production version should also include:

- Amazon Cognito or another trusted identity provider
- JWT-based authentication
- API Gateway authorizers
- Strong request validation
- Structured Lambda logging
- CloudWatch metric filters for denied access attempts
- AWS WAF rate limiting
- Automated authorization tests
- Centralized SIEM alerting
- Transaction-level authorization checks

These controls would strengthen the system, but they would not replace backend ownership validation.

Authentication identifies the requester.  
Authorization decides what the requester can access.

---

## Key Takeaway

The defense phase proved that the BOLA vulnerability could be remediated without breaking legitimate functionality.

The secure Lambda function still returns account data when the requester owns the account, but it blocks access when the requester attempts to access another user's account.
