## API Endpoints

- POST /login
- GET /accounts/{accountId}
- GET /accounts/{accountId}/transactions
- POST /transfer

## Data Model

User:
- userId
- username
- passwordHash (or Cognito-managed authentication)
- ownedAccountIds (list of account IDs the user is allowed to access)

Account:
- accountId
- ownerUserId
- accountType
- balance

Transaction:
- transactionId
- accountId
- amount
- type (deposit, withdrawal, transfer)
- timestamp

Transfer Request:
- sourceAccountId
- destinationAccountId
- amount

## Vulnerability Details

Broken Object Level Authorization (BOLA):
- The API does not verify that the authenticated user owns the requested accountId
- Any authenticated user can access another user's account data by modifying the accountId in the request path

Example:
- User A sends: GET /accounts/123
- User A changes the request to: GET /accounts/456
- The API returns account 456 data even though User A does not own that account

Missing Rate Limiting:
- The API does not restrict how frequently a user can send requests
- An attacker can send repeated requests to sensitive endpoints without being blocked or throttled

## Attack Flow

1. Authenticate as User A
2. Send a request to GET /accounts/123
3. Modify the request to GET /accounts/456
4. Receive unauthorized account data for another user
5. Send repeated requests to GET /accounts/456 or GET /accounts/456/transactions
6. Observe that the API does not throttle, block, or rate limit the activity

## Test Users

- User A owns account 123
- User B owns account 456
