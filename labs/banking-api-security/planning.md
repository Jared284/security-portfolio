## API Endpoints

- POST /login
- GET /accounts/{accountId}
- GET /accounts/{accountId}/transactions
- POST /transfer

## Data Model

User:
- userId
- username
- password (or Cognito-managed)
- accountIds (list of owned accounts)

Account:
- accountId
- ownerUserId
- balance

Transaction:
- transactionId
- accountId
- amount

## Vulnerability Details

Broken Authorization:
- API does NOT verify that the authenticated user owns the requested accountId
- Any authenticated user can access any account by modifying the accountId in the request

Example:
User A requests:
GET /accounts/123

Then modifies:
GET /accounts/456

API returns account 456 even though it belongs to another user
- type (deposit/withdraw/transfer)
- timestamp

## Attack Flow

1. Authenticate as User A
2. Send request to /accounts/{accountId}
3. Modify accountId to another user’s ID
4. Receive unauthorized account data
5. Send repeated requests to /accounts endpoint
6. Observe no rate limiting or blocking
