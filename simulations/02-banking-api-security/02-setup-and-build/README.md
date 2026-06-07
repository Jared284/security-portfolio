# Setup and Build

## Overview

This section documents how the vulnerable serverless banking API was built in AWS.

The goal of this phase was to create a functioning API that could retrieve account records from DynamoDB through Lambda and API Gateway. The initial version was intentionally built without object-level authorization so the BOLA vulnerability could be demonstrated later.

The build followed this sequence:

~~~text
DynamoDB Data Model
        ↓
Account Records Created
        ↓
Lambda Function Built
        ↓
IAM Permissions Added
        ↓
API Gateway Route Created
        ↓
API Requests Tested
        ↓
Vulnerable Behavior Confirmed
~~~

---

## Build Objectives

This setup phase proves that the backend system was real and functional before exploitation.

The build demonstrates:

- Creation of a banking-style data model
- DynamoDB table setup for account records
- Lambda backend implementation for account lookup
- IAM permission troubleshooting for DynamoDB access
- API Gateway route configuration
- Successful API requests through the public endpoint
- Vulnerable account access behavior before remediation

---

## AWS Services Used

- Amazon API Gateway
- AWS Lambda
- Amazon DynamoDB
- AWS IAM
- Amazon CloudWatch Logs

---

## Component Build Order

### 1. Data Model

The lab uses a simplified banking data model.

Core account fields:

| Field | Purpose |
|---|---|
| `accountId` | Unique account identifier used in the API path |
| `ownerUserId` | User who owns the account |
| `accountType` | Type of account, such as checking or savings |
| `balance` | Simulated account balance |

Example ownership model:

| Account ID | Owner User ID | Security Meaning |
|---|---|---|
| `acc-123` | `user-001` | Account owned by User 1 |
| `acc-456` | `user-002` | Account owned by User 2 |

The ownership field is critical because the secure version of the API must verify that the requester owns the requested account.

---

### 2. DynamoDB Setup

DynamoDB stores the account records used by the API.

The account table was configured so the backend Lambda function could look up accounts by `accountId`.

Expected behavior:

~~~text
Input: accountId
        ↓
DynamoDB lookup
        ↓
Return matching account record
~~~

Security note:

DynamoDB returning the correct record does not mean the requester is authorized to see it. Authorization must be enforced in the application logic before data is returned to the client.

Evidence:

- `transactions-records.png`

---

### 3. Vulnerable Lambda Function

The first Lambda function implemented the account lookup logic.

In the vulnerable version, the function:

1. Extracted `accountId` from the API request path.
2. Queried DynamoDB for that account.
3. Returned the account record.
4. Did not validate whether the requester owned the account.

Vulnerable logic:

~~~text
If accountId exists:
    return account data
~~~

This is insecure because the user controls the `accountId` value.

A secure API needs this logic instead:

~~~text
If accountId exists AND requester owns account:
    return account data
Else:
    deny access
~~~

Documentation:

- [Vulnerable Lambda Implementation](./lambda-get-account-vulnerable.md)

Evidence:

- `lambda-code-get-account.png`
- `lambda-test-acc-123.png`
- `lambda-test-acc-456.png`

---

### 4. IAM Permission Troubleshooting

During setup, the Lambda function initially failed because it did not have the required DynamoDB permissions.

This was a useful build issue because it showed the need for correct IAM permissions between Lambda and DynamoDB.

The Lambda execution role needed permission to read account records from DynamoDB.

Evidence:

- `lambda-permission-error.png`

Security note:

This is separate from the BOLA vulnerability. IAM controls whether Lambda can access DynamoDB. Application authorization controls whether the requester should receive a specific account record.

Both layers matter.

---

### 5. API Gateway Setup

API Gateway was configured to expose the Lambda function through an HTTP endpoint.

Route used:

~~~http
GET /accounts/{accountId}
~~~

The route accepts an `accountId` path parameter and forwards the request to the Lambda backend.

Request flow:

~~~text
Client request
        ↓
API Gateway route
        ↓
Lambda integration
        ↓
DynamoDB lookup
        ↓
API response
~~~

Evidence:

- `api-gateway-route.png`
- `api-gateway-request.png`

---

### 6. API Request Testing

After API Gateway was connected to Lambda, the endpoint was tested with account requests.

Expected normal request:

~~~http
GET /accounts/acc-123
~~~

Expected vulnerable request:

~~~http
GET /accounts/acc-456
~~~

In the vulnerable version, both requests returned account data because the backend did not enforce ownership validation.

Evidence:

- `bola-request-acc-123.png`
- `bola-request-acc-456.png`

---

## Vulnerable State Summary

At the end of the setup phase, the system had a working API with an intentional authorization weakness.

The vulnerable state was:

| Layer | Status | Security Meaning |
|---|---|---|
| API Gateway | Route exposed | Client can request account objects |
| Lambda | Retrieves account data | Backend functionality works |
| DynamoDB | Stores account records | Account data is available |
| IAM | Lambda can read DynamoDB | Service-to-service access works |
| Authorization Logic | Missing ownership check | BOLA vulnerability exists |

The system was functional, but insecure.

---

## Why the Vulnerability Exists

The vulnerable Lambda function treated possession of an `accountId` as enough to retrieve the account.

That is the core design flaw.

The API answered:

~~~text
Does this account exist?
~~~

But it failed to answer:

~~~text
Is this requester allowed to access this account?
~~~

That missing second question created the Broken Object Level Authorization vulnerability.

---

## Evidence Collected

Evidence from this setup phase includes:

| Screenshot | What It Shows |
|---|---|
| `api-gateway-route.png` | API Gateway route for account lookup |
| `api-gateway-request.png` | API request testing through Gateway |
| `lambda-code-get-account.png` | Vulnerable Lambda account lookup logic |
| `lambda-permission-error.png` | IAM permission issue during Lambda setup |
| `lambda-test-acc-123.png` | Lambda successfully returning account 123 |
| `lambda-test-acc-456.png` | Lambda successfully returning account 456 |
| `transactions-records.png` | Backend test records used for the lab |

---

## Build Result

The setup phase produced a working AWS serverless API that could retrieve banking-style account data from DynamoDB.

The system was intentionally left in a vulnerable state so the attack phase could demonstrate how a requester could modify the `accountId` path parameter and access another user's account data.

Next sections:

- [Operations and Commands](../03-operations-and-commands/README.md)
- [BOLA Attack Simulation](../05-attacks-and-simulation/bola-attack.md)
