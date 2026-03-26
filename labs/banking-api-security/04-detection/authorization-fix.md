# Authorization Fix

## Overview

The original implementation allowed unrestricted access to account data based solely on the accountId.

This fix introduces an ownership validation check to ensure that only the account owner can access the data.

## Changes Implemented

- Added user identity extraction from request headers
- Compared requester identity against account owner
- Denied access when identities do not match

## Updated Logic

Access is now granted only if:

- The account exists
- The requester identity matches the account owner

Otherwise, the request is rejected with a 403 Unauthorized response.

## Security Impact

This change prevents unauthorized access to other users’ accounts and mitigates the Broken Object Level Authorization (BOLA) vulnerability.

## Validation After Remediation

After implementing the ownership check in the Lambda function, the same requests used in the BOLA attack were re-tested.

### Authorized Request

A request was made using a valid user identity that matches the account owner:

- Endpoint: /accounts/acc-123  
- Header: x-user-id: user-001  

The API returned the account data successfully.

![Authorized Request](../screenshots/defenses/authorized-request.png)

---

### Unauthorized Request Attempt

The same user identity was used to request a different account:

- Endpoint: /accounts/acc-456  
- Header: x-user-id: user-001  

The API correctly denied access and returned an unauthorized response.

![Blocked Request](../screenshots/defenses/blocked-request.png)

---

## Result

The previous BOLA attack is no longer successful.

Access to account data is now restricted based on ownership, and unauthorized requests are blocked.
