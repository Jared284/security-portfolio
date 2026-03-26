## Evidence

### Permission Error

Initial execution failed due to missing DynamoDB permissions.

![Permission Error](../screenshots/lambda/lambda-permission-error.png)

---

### Lambda Code

The following implementation retrieves account data based solely on the provided accountId.

![Lambda Code](../screenshots/lambda/lambda-code-get-account.png)

---

### Test Result - Account 123

This confirms normal functionality for a valid account.

![Account 123](../screenshots/lambda/lambda-test-acc-123.png)

---

### Test Result - Account 456

This shows the function also returns data for a different account when requested directly.

![Account 456](../screenshots/lambda/lambda-test-acc-456.png)
