# Reflections and Improvements

## Overview

This section summarizes key takeaways from building and securing the banking API, along with potential improvements for a more production-ready system.

---

## Key Lessons Learned

See detailed breakdown:

- [Lessons Learned](./lessons-learned.md)

---

## Core Security Insight

The primary vulnerability in this lab was **Broken Object Level Authorization (BOLA)**.

The system originally trusted user input (`accountId`) without verifying ownership, allowing unauthorized access to sensitive data.

Fixing this required:

- Introducing identity into the request
- Enforcing ownership validation in backend logic
- Returning proper error responses when access is denied

---

## Why This Matters

This pattern is extremely common in real-world APIs.

Without proper authorization checks:

- Any authenticated user can access other users’ data
- Sensitive financial or personal data can be exposed
- The system becomes fundamentally insecure despite working “correctly”

---

## Improvements for Production Systems

To make this system more realistic and secure:

- Replace header-based identity with JWT authentication (Cognito)
- Add IAM-based authorization or API Gateway authorizers
- Implement logging and monitoring (CloudWatch, SIEM)
- Add rate limiting and throttling
- Validate all input parameters
- Introduce structured error handling

---

## Final Takeaway

Security is not just about building functionality.

A system can work perfectly and still be completely vulnerable.

Authorization must be explicitly enforced at every access point.
