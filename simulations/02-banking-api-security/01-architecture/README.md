# Architecture

## Overview

This lab uses a small serverless AWS architecture to simulate a banking API with an object-level authorization flaw.

The system was intentionally designed with three separate layers:

- **Public API access layer**: Amazon API Gateway
- **Application logic layer**: AWS Lambda
- **Data storage layer**: Amazon DynamoDB

This separation makes the authorization failure easier to analyze. The vulnerability is not in DynamoDB or API Gateway by themselves. The failure occurs in the application logic because the Lambda function originally trusts a user-controlled `accountId` without verifying that the requester owns the account.

---

## Architecture Diagram

~~~text
Client / Tester
        |
        | HTTP request
        | GET /accounts/{accountId}
        | Header: x-user-id
        v
Amazon API Gateway
        |
        | forwards request to Lambda
        v
AWS Lambda: getAccount
        |
        | queries account by accountId
        v
Amazon DynamoDB
Accounts Table
        |
        | returns account record
        v
AWS Lambda
        |
        | returns API response
        v
Client / Tester
~~~

---

## AWS Components

### Amazon API Gateway

API Gateway exposes the banking API endpoint to the client.

Endpoint used in this lab:

~~~http
GET /accounts/{accountId}
~~~

Responsibilities:

- Receive HTTP requests
- Accept the `accountId` path parameter
- Forward requests to the backend Lambda function
- Return Lambda responses to the client

Evidence:

- `api-gateway-route.png`
- `api-gateway-request.png`

---

### AWS Lambda: `getAccount`

The Lambda function contains the backend logic for retrieving account data.

In the vulnerable version, the function:

- Extracted `accountId` from the request path
- Queried DynamoDB using the supplied `accountId`
- Returned the account record
- Did not validate account ownership

In the secure version, the function:

- Extracts `accountId` from the request path
- Extracts requester identity from the `x-user-id` header
- Queries DynamoDB for the account record
- Compares the requester identity to the account owner
- Allows access only when ownership matches
- Returns `403 Unauthorized` when ownership does not match

Evidence:

- `lambda-code-get-account.png`
- `lambda-authorization-check.png`
- `lambda-test-acc-123.png`
- `lambda-test-acc-456.png`

---

### Amazon DynamoDB

DynamoDB stores the banking account records used by the API.

The account table uses `accountId` as the primary lookup value.

Example account ownership model:

| Account ID | Owner User ID | Purpose |
|---|---|---|
| `acc-123` | `user-001` | Account owned by User 1 |
| `acc-456` | `user-002` | Account owned by User 2 |

This ownership relationship is what the secure Lambda function must enforce.

Evidence:

- `transactions-records.png`

---

## Data Flow: Vulnerable Version

In the vulnerable version, the API trusted the `accountId` supplied by the requester.

~~~text
Requester supplies accountId
        ↓
API Gateway forwards request
        ↓
Lambda extracts accountId
        ↓
Lambda queries DynamoDB
        ↓
DynamoDB returns account record
        ↓
Lambda returns account data
        ↓
No ownership validation occurs
~~~

Vulnerable access pattern:

~~~text
GET /accounts/acc-123  → returns account 123
GET /accounts/acc-456  → returns account 456
~~~

The issue is that the requester can control which object is retrieved. If the backend does not verify ownership, the requester can access another user's account by changing the path parameter.

---

## Data Flow: Secure Version

In the secure version, the API uses both the requested object and the requester identity before returning data.

~~~text
Requester supplies accountId
        +
Requester identity from x-user-id header
        ↓
API Gateway forwards request
        ↓
Lambda queries DynamoDB for account
        ↓
Lambda compares requester identity to account owner
        ↓
If owner matches: return account data
If owner does not match: return 403 Unauthorized
~~~

Secure access pattern:

~~~text
user-001 → GET /accounts/acc-123  → allowed
user-001 → GET /accounts/acc-456  → denied
~~~

This enforces object-level authorization.

Evidence:

- `authorized-request.png`
- `blocked-request.png`

---

## Trust Boundaries

### 1. External Boundary: Client to API Gateway

The client controls the HTTP request.

User-controlled inputs include:

- Requested path
- `accountId`
- Request headers
- Simulated `x-user-id` identity header

Security concern:

- Any value supplied by the client must be treated as untrusted.

---

### 2. Application Boundary: API Gateway to Lambda

API Gateway passes request data into Lambda.

Security concern:

- Lambda must not assume that path parameters are authorized.
- Lambda must validate whether the requester is allowed to access the requested object.

---

### 3. Data Boundary: Lambda to DynamoDB

Lambda has permission to query account data from DynamoDB.

Security concern:

- DynamoDB may return the requested record correctly, but correctness is not the same as authorization.
- The application must decide whether the requester should receive the returned data.

---

## Vulnerability Location

The BOLA vulnerability exists in the Lambda authorization logic.

The insecure design was:

~~~text
accountId exists → return account data
~~~

The secure design is:

~~~text
accountId exists + requester owns account → return account data
~~~

This distinction is the core security lesson of the lab.

---

## Why This Architecture Matters

Serverless applications still require explicit authorization logic.

API Gateway can expose the endpoint. Lambda can process the request. DynamoDB can return the correct data. But if the application does not verify ownership, the system can still expose sensitive financial data.

In a banking or fintech system, every request for account data must answer two questions:

1. Does the requested account exist?
2. Is this requester authorized to access that account?

The vulnerable version answered only the first question.  
The secure version answers both.

---

## Architecture Limitations

This lab intentionally keeps the architecture small so the authorization flaw is easy to understand.

Current limitations:

- `x-user-id` is used as a simulated identity source
- No production-grade Cognito or JWT authorizer is implemented
- Only the account lookup endpoint is fully tested
- No transaction-level authorization is fully implemented
- No WAF or throttling layer is included in this architecture
- No centralized SIEM pipeline is included

In production, requester identity should come from a trusted authentication system such as Amazon Cognito, SAML/OIDC federation, or another JWT-based identity provider. A client-controlled identity header should not be trusted by itself.
