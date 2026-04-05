# Defense

## Overview

This section documents how the vulnerable banking API was remediated after the BOLA vulnerability was identified and exploited.

The fix focuses on enforcing object-level authorization to ensure that only the account owner can access account data.

---

## Remediation Strategy

The original system trusted user-controlled input (`accountId`) and returned data directly from DynamoDB.

The fix introduces:

- requester identity extraction (`x-user-id`)
- ownership validation against stored account data
- conditional access control (allow / deny)

---

## Components

### Authorization Fix

Explains the remediation logic, validation testing, and security impact.

→ [View Authorization Fix](./authorization-fix.md)

---

### Secure Lambda Implementation

Shows the updated Lambda function with ownership validation logic added.

→ [View Secure Lambda Implementation](./lambda-get-account-secure.md)

---

## Result

After remediation:

- access is no longer based solely on object identifiers
- requests require a user identity
- unauthorized access attempts are blocked

The previously demonstrated BOLA attack is no longer successful.
