# AWS Banking API Security Lab: Attack, Detect, Defend

## Overview

This lab demonstrates a real-world API security vulnerability (Broken Object Level Authorization - BOLA) in a cloud-based banking system, followed by a full remediation and validation process.

The system was intentionally built in an insecure state, exploited using a controlled attack, and then hardened using authorization logic to prevent unauthorized access.

---

## Architecture

- API Gateway (public API endpoint)
- AWS Lambda (backend logic)
- DynamoDB (account and transaction data)

---

## Vulnerability

The API returned account data based solely on a user-controlled identifier:

GET /accounts/{accountId}

No ownership validation was performed, allowing unauthorized access to other users' accounts.

---

## Attack

An attacker modified the accountId in the request:

- /accounts/acc-123 → legitimate
- /accounts/acc-456 → unauthorized access

The API returned data for both requests.

This demonstrates a Broken Object Level Authorization (BOLA) vulnerability.

---

## Remediation

The Lambda function was updated to:

- extract user identity from request headers
- compare identity against account ownership
- deny access when ownership does not match

---

## Validation

The same attack was re-tested:

- valid request → allowed
- modified request → blocked (403 Unauthorized)

This confirms the vulnerability was successfully mitigated.

---

## Key Takeaways

- Object-level authorization must be enforced server-side
- User-controlled identifiers cannot be trusted
- Security must be validated through testing, not assumptions
- Cloud-native systems require explicit access control at every layer
