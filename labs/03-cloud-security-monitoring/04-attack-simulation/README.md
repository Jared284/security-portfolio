# 04 - Attack Simulation

## Objective

The goal of this phase was to generate repeated unauthorized SSH login activity against the EC2 monitoring host in order to produce real authentication failure telemetry for the lab’s host-based detection pipeline.

This simulation was designed to test whether invalid SSH login attempts would:

- reach the EC2 instance
- generate authentication log entries in `/var/log/auth.log`
- support downstream detection and alerting in CloudWatch

## Attack Scenario

I simulated an external attacker attempting to access the EC2 instance over SSH using usernames that did not exist on the target Linux host.

Instead of trying valid usernames with incorrect passwords, I intentionally used fake usernames to generate `Invalid user` log entries. This mattered because the detection logic for this phase was built around matching those specific authentication events.

## Target

- **Target host:** `cloud-security-monitoring-host`
- **Target type:** AWS EC2 Ubuntu instance
- **Exposed service:** SSH on port `22`
- **Public IP used during testing:** `44.201.91.146`

## Attack Execution

From my local workstation, I generated repeated SSH login attempts using fake usernames:

```bash
ssh fakeadmin@44.201.91.146
ssh backupsvc@44.201.91.146
ssh tempops@44.201.91.146
ssh nobody123@44.201.91.146
ssh svc-test@44.201.91.146
```

Each attempt failed with a `Permission denied (publickey)` response, confirming that the requests reached the target host and were rejected by the SSH service.

## Why This Test Was Chosen

This test was selected because it produces realistic host-level authentication telemetry without requiring a successful compromise.

It also maps directly to common real-world reconnaissance and access-attempt behavior, where attackers try invalid or guessed usernames against exposed SSH services.

For this lab, the simulation was useful because it created a clear, repeatable signal that could be tracked across multiple layers:

- attacker terminal output
- Linux authentication logs
- centralized CloudWatch logs
- custom CloudWatch metrics
- CloudWatch alarming
- SNS notification delivery

## Validation

After generating the SSH attempts, I confirmed that the EC2 instance recorded the activity in `/var/log/auth.log`.

Example log entries included:

```text
Invalid user backupsvc from 128.6.147.103
Invalid user tempops from 128.6.147.103
Invalid user nobody123 from 128.6.147.103
Invalid user svc-test from 128.6.147.103
```

This confirmed that the simulated SSH activity produced the exact host-level telemetry needed for the next phases of detection engineering and alerting.

## Evidence

### Local Attack Simulation

![Repeated invalid-user SSH attempts from the local workstation](../screenshots/ssh-invalid-user-attempts-terminal.png)

### Host Log Validation

![EC2 auth.log entries showing invalid-user SSH attempts](../screenshots/auth-log-invalid-user-events.png)

## Key Takeaway

This phase established the initial signal for the host-monitoring side of the lab. It proved that externally generated SSH access attempts against the EC2 instance created real Linux authentication events that could be used as the foundation for centralized detection and alerting in AWS.
