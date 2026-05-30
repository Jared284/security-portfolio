# DynamoDB Setup

## Overview

This file documents how DynamoDB was used as the backend data store for the AWS Banking API Security Lab.

DynamoDB stored the simulated banking records used by the API. The table data created the ownership relationship needed to test Broken Object Level Authorization (BOLA).

The goal was to create separate account records so the API could be tested in two states:

1. A vulnerable state where any requested account could be returned
2. A secure state where only the account owner could access the account

---

## Role of DynamoDB in This Lab

DynamoDB was responsible for storing the banking data used by the Lambda backend.

The Lambda function queried DynamoDB using the `accountId` supplied in the API request.

Basic data flow:

~~~text
API request contains accountId
        ↓
Lambda extracts accountId
        ↓
Lambda queries DynamoDB
        ↓
DynamoDB returns matching account record
        ↓
Lambda decides whether to return or deny the response
~~~

DynamoDB returned the requested record correctly. The security issue came from whether Lambda should return that record to the requester.

---

## Table Purpose

The DynamoDB table was used to store simulated account records.

The table needed to support:

- Account lookup by `accountId`
- Multiple users with different accounts
- Ownership validation using `ownerUserId`
- Controlled BOLA testing by changing the requested object identifier

The key security relationship was:

~~~text
account.ownerUserId == requesterUserId
~~~

If that relationship is not checked, the API is vulnerable.

---

## Account Table Structure

The account table used `accountId` as the main lookup value.

| Field | Purpose |
|---|---|
| `accountId` | Unique identifier for the account |
| `ownerUserId` | User who owns the account |
| `accountType` | Simulated account type |
| `balance` | Simulated account balance |

Example records:

| Account ID | Owner User ID | Account Type | Security Meaning |
|---|---|---|---|
| `acc-123` | `user-001` | Checking | Should only be accessible by `user-001` |
| `acc-456` | `user-002` | Savings | Should only be accessible by `user-002` |

---

## Example DynamoDB Items

Example item for User 1:

~~~json
{
  "accountId": "acc-123",
  "ownerUserId": "user-001",
  "accountType": "checking",
  "balance": 1500
}
~~~

Example item for User 2:

~~~json
{
  "accountId": "acc-456",
  "ownerUserId": "user-002",
  "accountType": "savings",
  "balance": 9000
}
~~~

These records were intentionally simple so the authorization logic could be clearly tested.

---

## Why Multiple Accounts Were Needed

The BOLA vulnerability requires at least two separate account objects.

If the lab only had one account, there would be no meaningful way to prove unauthorized object access.

The two-account model allowed this test:

~~~text
Requester: user-001
Legitimate account: acc-123
Unauthorized target account: acc-456
~~~

The vulnerable API returned both accounts.

The secure API allowed `acc-123` and denied `acc-456`.

---

## Vulnerable DynamoDB Access Pattern

In the vulnerable version, Lambda queried DynamoDB directly using the `accountId` from the request.

Vulnerable pattern:

~~~text
GET /accounts/acc-456
        ↓
Lambda extracts acc-456
        ↓
Lambda queries DynamoDB for acc-456
        ↓
DynamoDB returns acc-456
        ↓
Lambda returns account data
        ↓
No ownership check occurs
~~~

The database lookup itself was not the vulnerability.

The vulnerability was returning the database result without checking whether the requester owned the returned account.

---

## Secure DynamoDB Access Pattern

In the secure version, Lambda still queries DynamoDB using the requested `accountId`, but it validates ownership before returning data.

Secure pattern:

~~~text
GET /accounts/acc-456
Header: x-user-id: user-001
        ↓
Lambda queries DynamoDB for acc-456
        ↓
DynamoDB returns account record
        ↓
Lambda compares:
requesterUserId = user-001
ownerUserId = user-002
        ↓
Mismatch detected
        ↓
Return 403 Unauthorized
~~~

This preserves backend functionality while preventing unauthorized data exposure.

---

## IAM Permission Requirement

The Lambda function needed permission to read from the DynamoDB table.

During setup, the Lambda function initially failed because the execution role did not have the required DynamoDB permissions.

This was fixed by giving the Lambda execution role permission to read the table.

Security distinction:

| Control | Purpose |
|---|---|
| IAM permission | Determines whether Lambda can access DynamoDB |
| Application authorization | Determines whether the requester can access a specific account |

The IAM fix allowed Lambda to query the table.

The authorization fix controlled whether the requester should receive the returned account data.

---

## Evidence

Screenshot evidence related to DynamoDB setup includes:

| Screenshot | What It Shows |
|---|---|
| `transactions-records.png` | Backend records used for account and transaction testing |
| `lambda-permission-error.png` | Lambda initially missing DynamoDB permissions |
| `lambda-test-acc-123.png` | Lambda successfully retrieving `acc-123` |
| `lambda-test-acc-456.png` | Lambda successfully retrieving `acc-456` |
| `bola-request-acc-123.png` | API request for the legitimate account |
| `bola-request-acc-456.png` | API request for a different account object |

---

## Security Observation

DynamoDB did exactly what it was supposed to do: return the item requested by the backend.

The security failure happened because the vulnerable Lambda function treated the returned data as safe to send back to the requester.

The insecure assumption was:

~~~text
If DynamoDB returns the account, send it to the user.
~~~

The secure assumption is:

~~~text
If DynamoDB returns the account, verify ownership before sending it to the user.
~~~

---

## Why DynamoDB Was Useful for This Lab

DynamoDB made the BOLA issue easy to demonstrate because each account was a separate object with a unique identifier.

That allowed the lab to clearly show:

- Object identifiers can be modified by the requester
- Backend systems may return valid data for modified identifiers
- Authorization must be enforced before returning sensitive data
- Ownership fields are required for object-level authorization

---

## Design Limitations

This DynamoDB setup was intentionally simplified.

Current limitations:

- The account records are simulated
- No real customer data is used
- The table is designed for lab testing, not production banking
- Transaction-level authorization is not fully implemented
- No encryption design review is included
- No backup, retention, or disaster recovery design is included
- No DynamoDB Streams detection pipeline is included

These limitations are acceptable for this lab because the purpose was to isolate and demonstrate object-level authorization failure.

---

## Key Takeaway

DynamoDB was the data source, but the BOLA vulnerability was an application authorization failure.

The backend could retrieve account records successfully. The missing control was verifying whether the requester was allowed to receive the specific account record returned by DynamoDB.
