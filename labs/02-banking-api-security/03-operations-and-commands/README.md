# Operations and Commands

## Overview

This section documents the commands, request formats, and testing patterns used to interact with the AWS Banking API Security Lab.

The purpose of this file is to show how the API was tested before and after remediation.

The same endpoint was used in both phases:

~~~http
GET /accounts/{accountId}
~~~

The difference was the backend authorization behavior.

---

## API Endpoint

The lab used the following account lookup route:

~~~http
GET /accounts/{accountId}
~~~

Example account requests:

~~~http
GET /accounts/acc-123
GET /accounts/acc-456
~~~

The `{accountId}` path parameter represents the account object being requested.

---

## Base URL

The API Gateway invoke URL follows this format:

~~~text
https://YOUR-API-ID.execute-api.YOUR-REGION.amazonaws.com
~~~

Example placeholder:

~~~text
https://YOUR-INVOKE-URL
~~~

Full example request format:

~~~text
https://YOUR-INVOKE-URL/accounts/acc-123
~~~

The real invoke URL should not be committed if the API is still active.

---

## Identity Header Used for Testing

The secure version of this lab uses a simulated requester identity header:

~~~http
x-user-id: user-001
~~~

This header represents the user making the request.

Important limitation:

~~~text
x-user-id is only used as a lab simplification.
~~~

In a production application, requester identity should come from a trusted authentication system such as Amazon Cognito, OIDC, SAML, or JWT claims. A client-controlled header should not be trusted by itself.

---

## Test Accounts

The lab used two simulated accounts:

| Account ID | Owner User ID | Expected Access |
|---|---|---|
| `acc-123` | `user-001` | `user-001` should be allowed |
| `acc-456` | `user-002` | `user-001` should be denied |

This setup allowed the same requester to test both legitimate and unauthorized object access.

---

## Vulnerable Request Pattern

In the vulnerable version, no ownership validation was enforced.

The requester could access account data by changing the object identifier in the path.

### Request for Legitimate Account

~~~bash
curl -X GET "https://YOUR-INVOKE-URL/accounts/acc-123"
~~~

Expected vulnerable result:

~~~text
200 OK
Account data returned
~~~

### Request for Different Account

~~~bash
curl -X GET "https://YOUR-INVOKE-URL/accounts/acc-456"
~~~

Expected vulnerable result:

~~~text
200 OK
Account data returned
~~~

Security meaning:

~~~text
The API returned account data based only on the supplied accountId.
~~~

This confirmed the Broken Object Level Authorization vulnerability.

---

## Vulnerable Behavior Summary

| Request | Expected Result | Security Meaning |
|---|---|---|
| `GET /accounts/acc-123` | Account data returned | Normal access works |
| `GET /accounts/acc-456` | Account data returned | Unauthorized object access succeeds |

The second request should not have been allowed for `user-001`.

---

## Secure Request Pattern

After remediation, requests included a simulated requester identity header.

### Authorized Request

~~~bash
curl -X GET "https://YOUR-INVOKE-URL/accounts/acc-123" \
  -H "x-user-id: user-001"
~~~

Expected secure result:

~~~text
200 OK
Account data returned
~~~

Security meaning:

~~~text
user-001 owns acc-123, so access is allowed.
~~~

---

### Unauthorized Request

~~~bash
curl -X GET "https://YOUR-INVOKE-URL/accounts/acc-456" \
  -H "x-user-id: user-001"
~~~

Expected secure result:

~~~text
403 Unauthorized
Access denied
~~~

Security meaning:

~~~text
user-001 does not own acc-456, so access is denied.
~~~

---

## Secure Behavior Summary

| Requester | Requested Account | Account Owner | Expected Result |
|---|---|---|---|
| `user-001` | `acc-123` | `user-001` | Allowed |
| `user-001` | `acc-456` | `user-002` | Denied |

The secure version preserved legitimate access while blocking unauthorized object access.

---

## Lambda Direct Testing

Before testing through API Gateway, the Lambda function was tested directly.

Direct Lambda testing helped confirm that:

- Lambda could receive an account ID
- Lambda could query DynamoDB
- DynamoDB returned account records
- IAM permissions were configured correctly
- The vulnerable logic returned any valid account record

Evidence:

| Screenshot | What It Shows |
|---|---|
| `lambda-test-acc-123.png` | Lambda returning account 123 |
| `lambda-test-acc-456.png` | Lambda returning account 456 |
| `lambda-permission-error.png` | Initial DynamoDB permission issue |

---

## API Gateway Testing

After Lambda testing worked, API Gateway was used to test the public endpoint.

API Gateway testing helped confirm that:

- The public route was configured correctly
- The `accountId` path parameter was passed to Lambda
- The API returned account data through the invoke URL
- The BOLA issue could be demonstrated through normal HTTP requests

Evidence:

| Screenshot | What It Shows |
|---|---|
| `api-gateway-route.png` | API Gateway route configuration |
| `api-gateway-request.png` | API Gateway request testing |
| `bola-request-acc-123.png` | Request for legitimate account |
| `bola-request-acc-456.png` | Modified request for another account |

---

## Before-and-After Validation

The same basic request pattern was tested before and after remediation.

| Test | Vulnerable Result | Secure Result |
|---|---|---|
| Request `acc-123` as `user-001` | Allowed | Allowed |
| Request `acc-456` as `user-001` | Allowed | Denied |
| Return account data from DynamoDB | Works | Works |
| Enforce ownership | No | Yes |

The key result is that the fix did not break the API. It changed the authorization decision.

---

## Expected HTTP Responses

### Successful Authorized Response

~~~text
HTTP 200 OK
~~~

Meaning:

~~~text
The requester is allowed to access the requested account.
~~~

---

### Unauthorized Response

~~~text
HTTP 403 Unauthorized
~~~

Meaning:

~~~text
The requester is identified but does not own the requested account.
~~~

---

### Missing or Invalid Account

~~~text
HTTP 404 Not Found
~~~

Meaning:

~~~text
The requested account does not exist.
~~~

---

## Testing Logic

The testing workflow followed this logic:

~~~text
1. Confirm normal account request works
        ↓
2. Modify accountId in the request path
        ↓
3. Confirm vulnerable version returns another account
        ↓
4. Add ownership validation
        ↓
5. Repeat the same modified request
        ↓
6. Confirm secure version blocks access
~~~

This proved both the vulnerability and the remediation.

---

## Security Interpretation

The vulnerable API relied on this unsafe assumption:

~~~text
If the requester knows a valid accountId, return the account.
~~~

The secure API uses this safer rule:

~~~text
Only return the account if the requester owns it.
~~~

This is the core difference between functional access and authorized access.

---

## Evidence Collected

Evidence related to operations and commands includes:

| Screenshot | What It Shows |
|---|---|
| `api-gateway-route.png` | Public API route for account lookup |
| `api-gateway-request.png` | API Gateway request testing |
| `bola-request-acc-123.png` | Request for allowed account |
| `bola-request-acc-456.png` | Modified request showing vulnerable object access |
| `authorized-request.png` | Authorized request after remediation |
| `blocked-request.png` | Unauthorized request blocked after remediation |
| `lambda-test-acc-123.png` | Direct Lambda test for account 123 |
| `lambda-test-acc-456.png` | Direct Lambda test for account 456 |

---

## Safety Notes

This lab used controlled test data only.

Operational safety practices:

- No real banking data was used
- No real customer data was stored
- API invoke URLs should be removed or sanitized if still active
- AWS account IDs and sensitive identifiers should be blurred in screenshots
- Access keys should never be committed
- Public API resources should be deleted or disabled when no longer needed

---

## Key Takeaway

The command and request testing proved the full security workflow.

The vulnerable version allowed object access based only on a user-controlled `accountId`. The secure version required both the requested object and the requester identity to match the account ownership record before returning data.
