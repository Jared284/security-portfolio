# Attacks and Simulation

## Overview

This section documents the controlled Broken Object Level Authorization (BOLA) attack performed against the AWS Banking API.

The goal of this phase was to prove that the vulnerable API allowed unauthorized access to another user's account data by modifying the `accountId` value in the request path.

The attack was performed in a controlled lab environment using simulated banking records and intentionally vulnerable backend logic.

---

## Attack Scenario

The vulnerable endpoint was:

~~~http
GET /accounts/{accountId}
~~~

The API returned account data based on the requested `accountId`.

In the vulnerable version, the backend did not verify whether the requester owned the requested account.

This allowed the following attack pattern:

~~~text
Request legitimate account
        ↓
Modify accountId in request path
        ↓
Request another user's account
        ↓
Account data returned
        ↓
BOLA confirmed
~~~

---

## Vulnerability Tested

The attack tested for Broken Object Level Authorization.

BOLA occurs when an API exposes object identifiers and fails to verify whether the requester is authorized to access the requested object.

In this lab:

| BOLA Element | Lab Example |
|---|---|
| Object | Bank account record |
| Object identifier | `accountId` |
| User-controlled input | `/accounts/{accountId}` |
| Unauthorized object | `acc-456` |
| Missing control | Ownership validation |
| Impact | Another user's account data returned |

---

## Test Accounts

The lab used two simulated accounts:

| User | Owned Account | Expected Access |
|---|---|---|
| `user-001` | `acc-123` | Should be allowed |
| `user-002` | `acc-456` | Should only be accessible by `user-002` |

The attack tested whether `user-001` could access `acc-456`.

---

## Attack Summary

The attack followed this sequence:

1. Send a normal request for `acc-123`
2. Confirm account data is returned
3. Modify the request path from `acc-123` to `acc-456`
4. Send the modified request
5. Observe that the vulnerable API returns data for `acc-456`
6. Document the unauthorized object access

The successful attack proved that the API trusted the object identifier without enforcing ownership.

---

## Attack Documentation

Detailed attack steps, evidence, and security analysis are documented here:

- [BOLA Attack Simulation](./bola-attack.md)

---

## Evidence Collected

Evidence from the attack phase includes:

| Screenshot | What It Shows |
|---|---|
| `bola-request-acc-123.png` | Request for the legitimate account |
| `bola-request-acc-456.png` | Modified request returning another account |
| `api-gateway-request.png` | API Gateway request testing |
| `lambda-test-acc-123.png` | Lambda retrieving account 123 |
| `lambda-test-acc-456.png` | Lambda retrieving account 456 |
| `transactions-records.png` | Backend records used for the test |

---

## Attack Result

The attack succeeded in the vulnerable version.

| Request | Expected Secure Result | Vulnerable Result |
|---|---|---|
| `GET /accounts/acc-123` | Allowed | Allowed |
| `GET /accounts/acc-456` | Denied | Allowed |

The second request confirmed the BOLA vulnerability because the API returned another user's account data.

---

## Security Impact

In a real banking or fintech application, this flaw could expose sensitive customer data.

Potential impact includes:

- Unauthorized account access
- Exposure of financial information
- Broken customer isolation
- Privacy violations
- Compliance risk
- Increased fraud potential if write actions are also vulnerable

---

## Remediation Link

The remediation is documented in the defense section:

- [Defense Overview](../04-defense/README.md)
- [Authorization Fix](../04-defense/authorization-fix.md)
- [Secure Lambda Implementation](../04-defense/lambda-get-account-secure.md)

---

## Key Takeaway

The attack phase proved that the vulnerable API allowed unauthorized object access.

Changing the URL from `/accounts/acc-123` to `/accounts/acc-456` should not allow a requester to access another user's account. The backend must verify ownership before returning sensitive data.
