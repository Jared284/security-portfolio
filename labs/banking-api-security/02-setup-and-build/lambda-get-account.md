## IAM Permission Issue

During initial testing, the Lambda function failed with an `AccessDeniedException` when attempting to read from DynamoDB.

This occurred because the Lambda execution role did not have permission to perform `dynamodb:GetItem`.

The issue was resolved by attaching the `AmazonDynamoDBFullAccess` policy to the Lambda execution role.

After applying the policy, the function successfully retrieved account data from the `Accounts` table.
