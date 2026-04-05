# Architecture

## Overview

This lab is built as a simple cloud-based API system designed to simulate a banking application.

The architecture intentionally separates:
- public access layer
- application logic
- data storage

This separation allows analysis of how vulnerabilities propagate across components.

---

## Components

### API Gateway
- Exposes the API publicly
- Routes incoming HTTP requests to backend services
- Endpoint: GET /accounts/{accountId}

### AWS Lambda (getAccount)
- Handles request processing
- Retrieves account data from DynamoDB
- Initially implemented without authorization checks

### DynamoDB
- Stores account and transaction data
- Accounts table keyed by accountId
- Transactions table keyed by accountId + transactionId

---

## Data Flow

1. Client sends request to:
   GET /accounts/{accountId}

2. API Gateway receives the request and forwards it to Lambda

3. Lambda extracts accountId from the request

4. Lambda queries DynamoDB using accountId

5. DynamoDB returns the account record

6. Lambda returns the response to the client

---

## Trust Boundaries

### External Boundary
- Internet → API Gateway
- No authentication in initial version
- Fully user-controlled input

### Application Boundary
- API Gateway → Lambda
- Request parameters passed directly into logic

### Data Boundary
- Lambda → DynamoDB
- Direct data retrieval based on request input

---

## Security Observation (Vulnerable State)

The system trusts user-supplied object identifiers (`accountId`) without verifying ownership.

This creates a direct path:

User Input → Lambda → Database → Response

With no authorization checks in between, enabling object-level access control bypass.
