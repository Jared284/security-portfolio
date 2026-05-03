# Attacks and Simulation

This section will document controlled misuse testing inside the lab environment.

The goal is not to attack AWS. The goal is to safely test what an over-permissioned identity could attempt and then validate that risky actions are denied after remediation.

Planned tests:

- Attempt access to intended resources
- Attempt access to unintended resources
- Attempt unnecessary S3 actions
- Attempt risky IAM actions
- Capture denied activity in CloudTrail
