# Data Model

## Overview

This file documents the simplified banking data model used in the AWS Banking API Security Lab.

The purpose of the data model was not to recreate a full banking database. The purpose was to create enough realistic account ownership structure to demonstrate a Broken Object Level Authorization (BOLA) vulnerability.

The key security rule is simple:

~~~text
A user should only be able to access accounts they own.
~~~

The vulnerable version of the API failed to enforce that rule.

---

## Data Model Purpose

The lab needed a data model that could support both normal access and unauthorized object access testing.

To make the BOLA vulnerability testable, the system needed at least two separate users and two separate accounts.

Example ownership relationship:

~~~text
user-001 owns acc-123
user-002 owns acc-456
~~~

If `user-001` can access `acc-456`, then the API has an object-level authorization failure.

---

## Core Entities

The simplified banking system uses three conceptual entities:

~~~text
User
 ↓
Account
 ↓
Transaction
~~~

For this lab, the most important relationship is:

~~~text
User → Account Ownership
~~~

The API must verify this relationship before returning account data.

---

## User Model

The user model represents the identity of the requester.

In this lab, requester identity is simulated using the `x-user-id` request header.

Example:

~~~http
x-user-id: user-001
~~~

User fields:

| Field | Purpose |
|---|---|
| `userId` | Unique identifier for the user |
| `username` | Human-readable username |
| `ownedAccountIds` | Accounts the user is allowed to access |

Example users:

| User ID | Example Role | Owned Account |
|---|---|---|
| `user-001` | Simulated banking customer | `acc-123` |
| `user-002` | Simulated banking customer | `acc-456` |

Security note:

In a production system, identity should not come from a client-controlled header. It should come from a trusted authentication provider such as Amazon Cognito, OIDC, SAML, or another JWT-based identity source.

The `x-user-id` header is used here only to isolate and test authorization logic.

---

## Account Model

The account model represents a banking account stored in DynamoDB.

Core fields:

| Field | Purpose |
|---|---|
| `accountId` | Unique account identifier |
| `ownerUserId` | User who owns the account |
| `accountType` | Simulated account type |
| `balance` | Simulated account balance |

Example account records:

| Account ID | Owner User ID | Account Type | Security Meaning |
|---|---|---|---|
| `acc-123` | `user-001` | Checking | Should only be accessible by `user-001` |
| `acc-456` | `user-002` | Savings | Should only be accessible by `user-002` |

The `ownerUserId` field is the critical security field.

Without checking `ownerUserId`, the API cannot determine whether the requester is allowed to access the account.

---

## DynamoDB Account Record Examples

Example item for `acc-123`:

~~~json
{
  "accountId": "acc-123",
  "ownerUserId": "user-001",
  "accountType": "checking",
  "balance": 1500
}
~~~

Example item for `acc-456`:

~~~json
{
  "accountId": "acc-456",
  "ownerUserId": "user-002",
  "accountType": "savings",
  "balance": 9000
}
~~~

These records create the ownership boundary used in the lab.

---

## Transaction Model

The transaction model represents activity associated with an account.

Example fields:

| Field | Purpose |
|---|---|
| `transactionId` | Unique transaction identifier |
| `accountId` | Account associated with the transaction |
| `amount` | Transaction amount |
| `type` | Deposit, withdrawal, or transfer |
| `timestamp` | Time of transaction |

Example transaction structure:

~~~json
{
  "transactionId": "txn-001",
  "accountId": "acc-123",
  "amount": 250,
  "type": "deposit",
  "timestamp": "2026-01-01T12:00:00Z"
}
~~~

Security note:

Transaction data creates the same authorization requirement as account data.

A requester should only be able to view transactions for accounts they own.

---

## API Object Identifier

The vulnerable API route uses `accountId` as a path parameter:

~~~http
GET /accounts/{accountId}
~~~

Example:

~~~http
GET /accounts/acc-123
~~~

In this request, the object identifier is:

~~~text
acc-123
~~~

A requester can modify this value:

~~~http
GET /accounts/acc-456
~~~

If the backend returns `acc-456` without checking ownership, the API is vulnerable to BOLA.

---

## Vulnerable Access Logic

The vulnerable implementation only checked whether the requested account existed.

Vulnerable logic:

~~~text
Request contains accountId
        ↓
Query DynamoDB for accountId
        ↓
If account exists, return account data
~~~

This is insecure because the API does not ask:

~~~text
Does the requester own this account?
~~~

The vulnerable logic treats knowledge of an object identifier as authorization to access the object.

That is the core BOLA failure.

---

## Secure Access Logic

The secure implementation checks both the requested object and the requester identity.

Secure logic:

~~~text
Request contains accountId
        +
Request contains requester identity
        ↓
Query DynamoDB for accountId
        ↓
Compare requester identity to ownerUserId
        ↓
If match: return account data
If mismatch: return 403 Unauthorized
~~~

Required authorization relationship:

~~~text
requesterUserId == account.ownerUserId
~~~

Example allowed request:

| Requester | Requested Account | Account Owner | Result |
|---|---|---|---|
| `user-001` | `acc-123` | `user-001` | Allowed |

Example denied request:

| Requester | Requested Account | Account Owner | Result |
|---|---|---|---|
| `user-001` | `acc-456` | `user-002` | Denied |

---

## BOLA Test Case

The data model supports a clear BOLA test.

### Legitimate Access

~~~text
Requester: user-001
Requested account: acc-123
Account owner: user-001
Expected result: allowed
~~~

### Unauthorized Access Attempt

~~~text
Requester: user-001
Requested account: acc-456
Account owner: user-002
Expected result: denied
~~~

In the vulnerable version, the unauthorized access attempt succeeded.

In the secure version, the unauthorized access attempt was blocked.

---

## Evidence

Screenshot evidence related to the data model includes:

| Screenshot | What It Shows |
|---|---|
| `transactions-records.png` | Backend records used for testing account/transaction access |
| `lambda-test-acc-123.png` | Lambda retrieving data for `acc-123` |
| `lambda-test-acc-456.png` | Lambda retrieving data for `acc-456` |
| `bola-request-acc-123.png` | API request for the legitimate account |
| `bola-request-acc-456.png` | Modified request for a different account |

---

## Security Importance

The data model is what makes authorization possible.

If account records do not include ownership data, the application has no reliable way to decide whether a requester should receive an account record.

A secure banking API needs both:

~~~text
Object identifier
+
Ownership relationship
~~~

The vulnerable API used only the object identifier.

The secure API used both.

---

## Design Limitation

This lab uses a simplified identity and account model.

Current limitations:

- `x-user-id` simulates identity instead of using Cognito or JWT claims
- Account records are simplified for lab purposes
- Transaction-level authorization is not fully implemented
- No real customer data is used
- No production authentication provider is connected

These limitations are intentional. The lab focuses on the authorization relationship between requester identity and requested account object.

---

## Key Takeaway

A secure API cannot rely on user-controlled object identifiers.

The backend must validate that the requester is authorized to access the object being requested. In this lab, that required comparing the requester identity to the `ownerUserId` stored with the account record.
