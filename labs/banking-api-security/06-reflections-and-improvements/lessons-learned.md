# Lessons Learned

## Security Design

- Authorization must be enforced at the object level, not just at the endpoint level
- Accepting user-controlled identifiers without validation creates direct exposure risks

## Implementation

- Cloud services (Lambda, API Gateway, DynamoDB) do not enforce security by default
- IAM permissions and application-level checks must be explicitly configured

## Attack Perspective

- Small input manipulations (changing an ID) can lead to significant data exposure
- APIs with predictable identifiers are highly susceptible to enumeration attacks

## Improvements

Future enhancements could include:

- replacing header-based identity with Cognito authentication
- adding rate limiting using AWS WAF
- implementing structured logging for detection visibility
- introducing transaction-level authorization checks
