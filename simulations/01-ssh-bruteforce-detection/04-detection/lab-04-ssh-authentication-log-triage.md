# Lab 04 – SSH Authentication Log Triage

## Objective

Investigate SSH authentication activity using Linux logs to determine whether the observed behavior represents benign user error, brute-force activity, username probing, or suspicious access behavior.

This lab focuses on SOC-style triage: reviewing raw telemetry, identifying meaningful patterns, and forming an analyst assessment.

---

## Triage Goal

The goal of this lab is to answer the question:

~~~text
Is this SSH authentication activity normal user behavior or suspicious attack behavior?
~~~

To answer that, the investigation looks at:

- failed password attempts
- invalid username attempts
- successful login events
- source IP repetition
- username patterns
- event timing
- Fail2Ban enforcement status

---

## Data Sources

| Source | Purpose |
|---|---|
| `/var/log/auth.log` | Primary SSH authentication telemetry |
| OpenSSH `sshd` events | Failed, invalid, and accepted login events |
| Fail2Ban `sshd` jail | Defensive response and banned IP validation |
| Windows PowerShell SSH output | Attacker-side validation |

Primary log source:

~~~text
/var/log/auth.log
~~~

---

## Environment

| Component | Value |
|---|---|
| Server OS | Ubuntu Server 24.04.3 LTS |
| Service | OpenSSH Server (`sshd`) |
| Defense Tool | Fail2Ban |
| Source IP | `192.168.56.1` |
| Target IP | `192.168.56.101` |
| Log Source | `/var/log/auth.log` |

---

## Initial Signal

The investigation began after repeated SSH authentication failures were observed from the same source IP.

Initial suspicious indicators:

- multiple failed SSH password attempts
- multiple invalid username attempts
- activity from a repeated source IP
- attempts against common usernames such as `admin`, `test`, and `root`

Evidence:

![Controlled Failed SSH Attempts](../screenshots/05-controlled-failed-ssh-attempts.png)

---

## Investigation Workflow

The triage workflow followed this process:

~~~text
Review failed SSH events
        ↓
Review invalid username events
        ↓
Aggregate failed attempts by source IP
        ↓
Check for successful logins
        ↓
Check Fail2Ban status
        ↓
Validate whether the source was blocked
        ↓
Form analyst assessment
~~~

This workflow mirrors how a SOC analyst moves from raw events to an investigation conclusion.

---

## Step 1 – Review Failed SSH Authentication Events

Command used:

~~~bash
sudo grep "sshd" /var/log/auth.log | grep "Failed password" | tail -20
~~~

Purpose:

- identify failed SSH authentication attempts
- confirm the source IP involved
- identify target usernames
- determine whether the failures were isolated or repeated

Evidence:

![Auth Log Failed Password Events](../screenshots/06-auth-log-failed-password-events.png)

Key observation:

~~~text
Failed password events were generated from 192.168.56.1.
~~~

---

## Step 2 – Review Invalid Username Attempts

Command used:

~~~bash
sudo grep "sshd" /var/log/auth.log | grep "Invalid user" | tail -20
~~~

Purpose:

- identify attempts against non-existent accounts
- detect username probing behavior
- determine whether the source was guessing common usernames

Evidence:

![Auth Log Invalid User Events](../screenshots/07-auth-log-invalid-user-events.png)

Key observation:

~~~text
The source IP attempted authentication against invalid usernames.
~~~

This is more suspicious than a normal mistyped password because it suggests account discovery or automated probing.

---

## Step 3 – Aggregate Failed Attempts by Source IP

Command used:

~~~bash
sudo grep "Failed password" /var/log/auth.log | grep -oP 'from \K[0-9]+\.[0-9]+\.[0-9]+\.[0-9]+' | sort | uniq -c | sort -nr
~~~

Purpose:

- count failed SSH attempts by source IP
- identify repeat offenders
- determine whether failures are concentrated from one system

Evidence:

![Failed Logins Aggregated by IP](../screenshots/08-failed-logins-aggregated-by-ip.png)

Key observation:

~~~text
Multiple failed SSH attempts were associated with 192.168.56.1.
~~~

This supports a brute-force-style interpretation rather than an isolated login mistake.

---

## Step 4 – Review Successful Login Activity

Command used:

~~~bash
sudo grep "sshd" /var/log/auth.log | grep "Accepted password" | tail -20
~~~

Purpose:

- identify whether any login succeeded
- establish a normal successful login baseline
- determine whether failed attempts later resulted in successful access

Evidence:

![Successful SSH Login After Unban](../screenshots/11-successful-ssh-login-after-unban.png)

A successful login by itself is not malicious. However, successful authentication becomes more important when it follows failed or invalid attempts from the same source IP.

That higher-risk pattern is expanded later in Lab 07.

---

## Step 5 – Check Fail2Ban Jail Status

Command used:

~~~bash
sudo fail2ban-client status sshd
~~~

Purpose:

- determine whether Fail2Ban detected repeated failures
- check the number of failed attempts
- verify whether a source IP was banned

Evidence:

![Fail2Ban Banned IP After Attack](../screenshots/09-fail2ban-banned-ip-after-attack.png)

Key observation:

~~~text
192.168.56.1 was added to the Fail2Ban sshd jail banned IP list.
~~~

This confirmed that the observed activity crossed the automated response threshold.

---

## Step 6 – Validate Attacker-Side Blocking

Command used from Windows PowerShell:

~~~powershell
ssh admin@192.168.56.101
~~~

Purpose:

- confirm that the Fail2Ban ban was actually enforced
- validate blocking from the attacker's perspective
- prove that the response worked beyond only checking the defender-side status output

Evidence:

![SSH Blocked After Fail2Ban Ban](../screenshots/10-ssh-blocked-after-fail2ban-ban.png)

Key observation:

~~~text
The SSH connection timed out after the source IP was banned.
~~~

---

## Triage Findings

| Finding | Evidence | Interpretation |
|---|---|---|
| Repeated failed logins | Failed password logs | Suspicious authentication behavior |
| Invalid usernames | Invalid user logs | Possible username probing |
| Same source IP | Source IP aggregation | Concentrated attack source |
| Fail2Ban ban | sshd jail status | Threshold exceeded |
| SSH timeout after ban | PowerShell test | Blocking was enforced |

---

## Analyst Assessment

The activity is consistent with SSH brute-force or credential probing behavior.

The strongest indicators were:

- repeated failed authentication attempts
- invalid username attempts
- activity from the same source IP
- Fail2Ban threshold being exceeded
- automated blocking of the source IP

This behavior is more suspicious than benign user error because the attempts targeted multiple invalid or common usernames and occurred repeatedly from one source.

---

## Severity Reasoning

This activity would likely be treated as low-to-medium severity in a controlled or internal environment, but it could become higher severity if:

- the SSH service were internet-facing
- the source IP were unknown or external
- a successful login followed the failed attempts
- the activity targeted privileged users
- similar attempts occurred across multiple hosts
- post-login activity appeared after authentication

In this lab, the activity was controlled and expected, but the log pattern itself reflects real SSH attack behavior.

---

## Benign vs Suspicious Comparison

| Behavior | Likely Benign | More Suspicious |
|---|---|---|
| One failed password | User typo | Not enough alone |
| Repeated failures | Possible user issue | Brute-force indicator |
| Invalid usernames | Rare for normal users | Strong probing signal |
| Multiple usernames | Unusual | Username enumeration |
| Same source IP repeatedly failing | Could be misconfigured client | Suspicious if repeated quickly |
| Failed attempts followed by success | Could be user typo | Possible compromise pattern |

---

## Response Outcome

Fail2Ban automatically responded to the repeated SSH failures.

Response actions:

- source IP was added to the `sshd` jail banned list
- firewall enforcement blocked future SSH connections from that source
- attacker-side SSH attempt timed out after the ban

This confirmed that detection signals successfully triggered automated mitigation.

---

## Limitations

This triage was performed in a controlled local lab.

Limitations include:

- one source IP
- one target host
- no centralized SIEM
- no user identity enrichment
- no GeoIP or ASN enrichment
- no endpoint telemetry beyond SSH logs
- no post-login command monitoring
- no long-term baseline of normal SSH behavior

These limitations are acceptable because this lab focuses on local SSH log triage and response validation.

---

## Analyst Takeaways

- Raw authentication logs are noisy until grouped by source IP, username, and time.
- Invalid username attempts are stronger suspicious signals than ordinary failed passwords.
- Repeated failures from the same source IP justify investigation.
- Fail2Ban status should be validated alongside raw logs.
- Defender-side status output should be confirmed with attacker-side testing.
- Failed-to-successful login correlation is a higher-value detection than failed login counting alone.

---

## Outcome

This lab demonstrated how to triage SSH authentication activity using Linux logs and Fail2Ban status output.

The investigation confirmed that repeated failed authentication attempts from `192.168.56.1` represented brute-force-style behavior and triggered automated mitigation.

---

## Key Takeaway

SOC triage is not just looking at one failed login.

The value comes from correlating source IP, username, timing, authentication result, and response status to decide whether the activity is benign or suspicious.
