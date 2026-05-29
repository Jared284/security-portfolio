# API Gateway Setup

## Overview

This file documents how Amazon API Gateway was used to expose the banking API endpoint.

API Gateway acted as the public entry point for the lab. It accepted HTTP requests from the client, captured the requested `accountId` path parameter, and forwarded the request to the `getAccount` Lambda function.

The API Gateway route was intentionally simple so the lab could focus on the backend authorization flaw.

---

## Role of API Gateway in This Lab

API Gateway was responsible for:

- Exposing a public API endpoint
- Defining the route for account lookup requests
- Accepting the `accountId` path parameter
- Forwarding requests to AWS Lambda
- Returning Lambda responses to the requester

API Gateway did not enforce object-level authorization in this lab.

That responsibility belonged to the backend Lambda function.

---

## Route Created

The API exposed the following route:

~~~http
GET /accounts/{accountId}
~~~

Example requests:

~~~http
GET /accounts/acc-123
GET /accounts/acc-456
~~~

The `{accountId}` value is controlled by the requester.

That is what made this endpoint useful for demonstrating Broken Object Level Authorization.

---

## Request Flow

~~~text
Client / Tester
        ↓
GET /accounts/{accountId}
        ↓
Amazon API Gateway
        ↓
AWS Lambda: getAccount
        ↓
Amazon DynamoDB
        ↓
Lambda response returned through API Gateway
        ↓
Client / Tester
~~~

---

## Path Parameter Behavior

The key input in this route is:

~~~text
accountId
~~~

This value is taken from the URL path.

Example:

~~~http
GET /accounts/acc-123
~~~

In this request, API Gateway passes the following value to Lambda:

~~~text
accountId = acc-123
~~~

Another request:

~~~http
GET /accounts/acc-456
~~~

passes:

~~~text
accountId = acc-456
~~~

The security issue is not that API Gateway accepts a path parameter. That is normal API behavior.

The security issue occurs when the backend trusts that path parameter without verifying whether the requester is authorized to access the requested account.

---

## Vulnerable API Behavior

In the vulnerable version, the route allowed a requester to control which account object was retrieved.

~~~text
Requester changes accountId
        ↓
API Gateway forwards request
        ↓
Lambda queries DynamoDB using accountId
        ↓
Account data is returned
        ↓
No ownership check occurs
~~~

This allowed the following insecure behavior:

| Request | Result |
|---|---|
| `GET /accounts/acc-123` | Account 123 returned |
| `GET /accounts/acc-456` | Account 456 returned |

Because no ownership validation was enforced, a requester could modify the object identifier and retrieve another user's account data.

---

## Evidence

Screenshots captured during this setup include:

| Screenshot | What It Shows |
|---|---|
| `api-gateway-route.png` | API Gateway route for `GET /accounts/{accountId}` |
| `api-gateway-request.png` | API Gateway request testing against the account endpoint |
| `bola-request-acc-123.png` | Request for the first account |
| `bola-request-acc-456.png` | Modified request for another account |

---

## Security Observation

API Gateway correctly routed the request to Lambda.

The vulnerability was not caused by broken routing. The route worked as configured.

The problem was that the backend Lambda function originally treated this as sufficient:

~~~text
Valid accountId supplied → return account data
~~~

A secure design requires this instead:

~~~text
Valid accountId supplied
        +
Requester owns account
        ↓
Return account data
~~~

This distinction is the core of the BOLA vulnerability.

---

## Why API Gateway Alone Was Not Enough

API Gateway can expose endpoints, route requests, and integrate with Lambda.

However, API Gateway does not automatically know whether a requester should be allowed to access a specific banking account record.

For this lab, the important security decision had to happen inside the application logic:

~~~text
Does this requester own the account they are asking for?
~~~

Without that check, the API could be fully functional and still insecure.

---

## Secure Design Consideration

In the remediated version, API Gateway still forwards the request to Lambda, but Lambda performs an ownership check before returning data.

Secure request flow:

~~~text
Client request
        ↓
API Gateway route
        ↓
Lambda extracts accountId
        ↓
Lambda extracts requester identity
        ↓
Lambda compares requester identity to account owner
        ↓
Allowed if owner matches
Denied if owner does not match
~~~

This means API Gateway remains the routing layer, while Lambda becomes the authorization enforcement point.

---

## Production Improvements

This lab used a simplified setup to focus on object-level authorization.

In a production system, API Gateway should be strengthened with additional controls such as:

- Amazon Cognito authentication
- JWT authorizers
- API Gateway Lambda authorizers
- Request validation
- Rate limiting and throttling
- AWS WAF protections
- Structured access logging
- CloudWatch metric filters for suspicious request patterns

However, even with those controls, backend object-level authorization would still be required.

Authentication proves who the requester is.  
Object-level authorization proves what data they are allowed to access.

---

## Key Takeaway

API Gateway successfully exposed the banking API route, but routing alone does not secure sensitive data.

The route allowed clients to request different account objects. The vulnerable backend returned those objects without verifying ownership, creating a Broken Object Level Authorization vulnerability.

The secure design keeps API Gateway as the public entry point but requires Lambda to enforce ownership before returning account data.
