# Build Order

## Overview

This file documents the order used to build the AWS Banking API Security Lab.

The build was intentionally completed in phases so the vulnerable behavior could be created, tested, exploited, remediated, and validated in a controlled way.

The goal was not to build a production banking platform. The goal was to build enough realistic AWS infrastructure to demonstrate a Broken Object Level Authorization vulnerability and prove the fix.

---

## High-Level Build Sequence

~~~text
1. Define banking data model
        ↓
2. Create DynamoDB records
        ↓
3. Build vulnerable Lambda function
        ↓
4. Add Lambda permissions for DynamoDB
        ↓
5. Create API Gateway route
        ↓
6. Test normal API functionality
        ↓
7. Confirm vulnerable object access
        ↓
8. Simulate BOLA attack
        ↓
9. Add authorization logic
        ↓
10. Re-test authorized and unauthorized requests
        ↓
11. Document evidence and limitations
~~~

---

## Phase 1: Define the Data Model

The first step was defining a simple banking-style data model.

The account records needed enough information to support an ownership check.

Core fields:

| Field | Purpose |
|---|---|
| `accountId` | Unique account identifier requested through the API |
| `ownerUserId` | User who owns the account |
| `accountType` | Simulated account type |
| `balance` | Simulated account balance |

Example relationship:

| Account ID | Owner User ID |
|---|---|
| `acc-123` | `user-001` |
| `acc-456` | `user-002` |

This ownership relationship is what makes the BOLA vulnerability testable.

---

## Phase 2: Create DynamoDB Records

After defining the data model, test account records were created in DynamoDB.

The records represented separate users with separate accounts.

Purpose of this phase:

- Store realistic account objects
- Create more than one account for authorization testing
- Support direct account lookup by `accountId`
- Provide backend data for Lambda to retrieve

Evidence:

- `transactions-records.png`

---

## Phase 3: Build the Vulnerable Lambda Function

Next, the first version of the `getAccount` Lambda function was created.

The vulnerable Lambda function performed the following steps:

1. Extract `accountId` from the request.
2. Query DynamoDB for the matching account.
3. Return the account data.
4. Do not check whether the requester owns the account.

Vulnerable logic:

~~~text
accountId supplied
        ↓
query DynamoDB
        ↓
return account data
~~~

This created the vulnerable state required for the BOLA attack.

Documentation:

- [Vulnerable Lambda Implementation](./lambda-get-account-vulnerable.md)

Evidence:

- `lambda-code-get-account.png`

---

## Phase 4: Fix Lambda-to-DynamoDB Permissions

During testing, the Lambda function initially failed because the execution role did not have the required DynamoDB permissions.

This was fixed by granting Lambda permission to read the required DynamoDB table.

Purpose of this phase:

- Allow Lambda to query DynamoDB
- Validate AWS service-to-service permissions
- Confirm the backend could retrieve account records

Evidence:

- `lambda-permission-error.png`

Security note:

This IAM issue is separate from the BOLA vulnerability.

IAM controls whether Lambda can access DynamoDB.  
Application authorization controls whether the requester should receive the returned account data.

Both controls are important, but they solve different problems.

---

## Phase 5: Test Lambda Directly

Before exposing the API through API Gateway, the Lambda function was tested directly.

Tests included:

| Test | Purpose |
|---|---|
| Request `acc-123` | Confirm Lambda could retrieve account 123 |
| Request `acc-456` | Confirm Lambda could retrieve account 456 |

The tests proved the Lambda function and DynamoDB lookup were working.

Evidence:

- `lambda-test-acc-123.png`
- `lambda-test-acc-456.png`

---

## Phase 6: Create API Gateway Route

After Lambda testing worked, API Gateway was configured to expose the function through an HTTP route.

Route:

~~~http
GET /accounts/{accountId}
~~~

Purpose of this phase:

- Expose the account lookup API
- Pass the `accountId` path parameter to Lambda
- Allow the endpoint to be tested like a real API

Documentation:

- [API Gateway Setup](./api-gateway.md)

Evidence:

- `api-gateway-route.png`
- `api-gateway-request.png`

---

## Phase 7: Confirm Normal API Functionality

Once API Gateway was connected to Lambda, normal requests were tested.

Example normal request:

~~~http
GET /accounts/acc-123
~~~

Expected result:

~~~text
Account data returned
~~~

This confirmed that the API was functional.

---

## Phase 8: Confirm Vulnerable Object Access

After confirming normal functionality, the object identifier was modified.

Example modified request:

~~~http
GET /accounts/acc-456
~~~

In the vulnerable version, the API returned account data for the modified account.

This confirmed the Broken Object Level Authorization condition.

Vulnerable behavior:

| Request | Result |
|---|---|
| `GET /accounts/acc-123` | Account 123 returned |
| `GET /accounts/acc-456` | Account 456 returned |

Evidence:

- `bola-request-acc-123.png`
- `bola-request-acc-456.png`

---

## Phase 9: Add Authorization Logic

After the vulnerable behavior was confirmed, the Lambda function was updated with an ownership check.

The secure Lambda function performs the following steps:

1. Extract `accountId` from the request path.
2. Extract requester identity from the `x-user-id` header.
3. Query DynamoDB for the requested account.
4. Compare `x-user-id` to the account's `ownerUserId`.
5. Return account data only if ownership matches.
6. Return `403 Unauthorized` if ownership does not match.

Secure logic:

~~~text
requested accountId
        +
requester identity
        ↓
lookup account
        ↓
compare requester to owner
        ↓
allow or deny
~~~

Documentation:

- [Secure Lambda Implementation](../04-defense/lambda-get-account-secure.md)
- [Authorization Fix](../04-defense/authorization-fix.md)

Evidence:

- `lambda-authorization-check.png`

---

## Phase 10: Validate the Fix

After remediation, the same access pattern was tested again.

Validation cases:

| Request | Header | Expected Result |
|---|---|---|
| `GET /accounts/acc-123` | `x-user-id: user-001` | Allowed |
| `GET /accounts/acc-456` | `x-user-id: user-001` | Denied |

The key validation point was that legitimate access still worked while unauthorized access was blocked.

Evidence:

- `authorized-request.png`
- `blocked-request.png`

---

## Phase 11: Document Evidence

After the build, attack, and remediation were complete, screenshots were organized and mapped to the documentation.

Evidence collected:

| Evidence | Purpose |
|---|---|
| `api-gateway-route.png` | Proves the API Gateway route was created |
| `api-gateway-request.png` | Shows API request testing |
| `lambda-code-get-account.png` | Shows vulnerable Lambda logic |
| `lambda-permission-error.png` | Shows IAM permission troubleshooting |
| `lambda-test-acc-123.png` | Shows Lambda retrieving account 123 |
| `lambda-test-acc-456.png` | Shows Lambda retrieving account 456 |
| `bola-request-acc-123.png` | Shows normal account request |
| `bola-request-acc-456.png` | Shows modified object request |
| `lambda-authorization-check.png` | Shows remediated authorization logic |
| `authorized-request.png` | Shows valid access after remediation |
| `blocked-request.png` | Shows unauthorized access blocked |
| `transactions-records.png` | Shows backend test records |

---

## Final Build State

At the end of the build process, the lab contained:

- A serverless AWS banking API
- DynamoDB account records with ownership fields
- A vulnerable Lambda implementation
- A public API Gateway account route
- Evidence of unauthorized object access
- A remediated Lambda implementation
- Evidence that unauthorized access was blocked after the fix

---

## Key Takeaway

The build process proves that the lab was not just a written scenario.

The API was built, tested, exploited, fixed, and re-tested. The vulnerable behavior came from a real AWS serverless workflow where the backend returned account data based on a user-controlled object identifier without validating ownership.
