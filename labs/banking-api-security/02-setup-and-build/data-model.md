## DynamoDB Tables

### Accounts Table
Primary Key: accountId (string)

Example Records:

{
  "accountId": "acc-123",
  "ownerUserId": "user-001",
  "accountType": "checking",
  "balance": 5000
}

{
  "accountId": "acc-456",
  "ownerUserId": "user-002",
  "accountType": "savings",
  "balance": 8000
}

---

### Transactions Table
Primary Key: transactionId (string)

Attributes:
- accountId (string)
- amount (number)
- type (string)
- timestamp (string)

Example Records:

{
  "transactionId": "tx-001",
  "accountId": "acc-123",
  "amount": -200,
  "type": "withdrawal",
  "timestamp": "2026-03-23T10:00:00Z"
}

{
  "transactionId": "tx-002",
  "accountId": "acc-456",
  "amount": 1000,
  "type": "deposit",
  "timestamp": "2026-03-23T11:00:00Z"
}

---

## Test Users (Logical Mapping)

User A:
- userId: user-001
- owns account: acc-123

User B:
- userId: user-002
- owns account: acc-456

Note:
To efficiently query transactions by accountId, a secondary index (GSI) may be used on accountId.
