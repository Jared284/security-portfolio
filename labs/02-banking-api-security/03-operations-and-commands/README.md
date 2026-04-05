# Operations and Commands

## Overview

This section provides the commands and request formats used to interact with the API during testing, attack simulation, and validation.

These examples demonstrate how the system behaves before and after the authorization fix.

---

## API Endpoint

GET /accounts/{accountId}

Base URL:

https://YOUR-INVOKE-URL

---

## Request Format

Requests require a user identity header:

x-user-id: <user-id>

---

## Example: Authorized Request

`curl -H "x-user-id: user-001" https://YOUR-INVOKE-URL/accounts/acc-123`

Expected result:
- 200 OK
- account data returned

---

## Example: Unauthorized Request

`curl -H "x-user-id: user-001" https://YOUR-INVOKE-URL/accounts/acc-456`

Expected result:
- 403 Unauthorized
- access denied

---

## Vulnerable Behavior (Pre-Fix)

`curl https://YOUR-INVOKE-URL/accounts/acc-456`

Result:
- account data returned with no identity required
- demonstrates unrestricted access

---

## Notes

- Requests can be tested using curl or Postman
- The x-user-id header simulates authenticated user identity
- This approach is used for controlled testing of authorization logic
