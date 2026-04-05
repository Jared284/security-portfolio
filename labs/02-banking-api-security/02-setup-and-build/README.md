# Setup and Build

## Overview

This section documents how the vulnerable banking API system was built in AWS.

The goal of this phase was to create a functioning cloud-based API with intentionally missing authorization controls, allowing for later exploitation and remediation.

Each component is built separately and then integrated into a complete system.

---

## Build Order

The system was built in the following order:

1. Data Model Definition  
2. DynamoDB Tables  
3. Lambda Function (getAccount)  
4. API Gateway Integration  

Detailed steps for each component are documented below.

---

## Components

### Data Model

Defines the structure of users, accounts, and transactions stored in DynamoDB.

→ [View Data Model](./data-model.md)

---

### DynamoDB Setup

Creates and populates the Accounts and Transactions tables used by the backend.

→ [View DynamoDB Setup](./dynamodb-setup.md)

---

### Lambda Function (getAccount)

Implements backend logic for retrieving account data.

Initial version:
- accepts accountId from request
- queries DynamoDB
- returns result with no authorization checks

→ [View Lambda Implementation](./lambda-get-account.md)

---

### API Gateway

Exposes the Lambda function through a public HTTP endpoint.

Endpoint:
GET /accounts/{accountId}

→ [View API Gateway Setup](./api-gateway.md)

---

## Vulnerable State Summary

At the end of this setup phase:

- The API is publicly accessible
- Requests are processed using user-controlled identifiers
- No authentication or authorization is enforced

This creates a direct path from user input to database access, enabling object-level access control bypass.

This vulnerable state is intentionally preserved for exploitation in the attack phase.
