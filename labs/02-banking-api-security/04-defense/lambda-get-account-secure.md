# Lambda Function: getAccount (Secure Implementation)

## Overview

This version of the `getAccount` Lambda function adds an authorization check before returning account data.

Unlike the initial vulnerable implementation, this version no longer trusts the requested `accountId` by itself. It validates that the requester is authorized to access the account.

---

## Behavior

- Extracts `accountId` from the request path
- Extracts user identity from the request header
- Queries DynamoDB for the requested account
- Compares the requester identity to the account owner
- Returns the account only if ownership matches

This changes the access flow from:

User Input → Database Query → Response

to:

User Input + Requester Identity → Ownership Validation → Conditional Response

---

## Security Improvement

The original implementation allowed unrestricted access to account data based solely on the requested object identifier.

The remediated implementation adds an object-level authorization check:

- If the requester owns the account, access is allowed
- If the requester does not own the account, access is denied

This mitigates the Broken Object Level Authorization (BOLA) vulnerability demonstrated in the attack phase.

---

## Evidence

### Updated Lambda Code

This screenshot shows the remediated implementation with ownership validation logic added before returning account data.

![Secure Lambda Code](../screenshots/defenses/lambda-authorization-check.png)

---

### Authorized Request

This request shows that a valid user can still access their own account after the authorization check was added.

![Authorized Request](../screenshots/defenses/authorized-request.png)

---

### Blocked Unauthorized Request

This request shows that access is denied when a user attempts to access another user's account.

![Blocked Request](../screenshots/defenses/blocked-request.png)

---

## Conclusion

The updated Lambda function enforces object-level authorization by validating account ownership before returning data.

This prevents unauthorized account access and successfully remediates the original vulnerable behavior.
