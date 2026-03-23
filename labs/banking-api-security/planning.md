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
- type (deposit/withdraw/transfer)
- timestamp
