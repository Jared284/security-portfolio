# Operations and Commands

## Overview

This section provides the commands and request formats used to interact with the API during testing, attack simulation, and validation.

These examples demonstrate how the system behaves before and after the authorization fix.

---

## API Endpoint

GET /accounts/{accountId}

Base URL:

[[YOUR-INVOKE-URL](https://ucu7ig9pl2.execute-api.us-east-1.amazonaws.com)](https://ucu7ig9pl2.execute-api.us-east-1.amazonaws.com)

---

## Request Format

Requests require a user identity header:

x-user-id: <user-id>

---

## Example: Authorized Request

```bash
curl -H "x-user-id: user-001" https://YOUR-INVOKE-URL/accounts/acc-123
