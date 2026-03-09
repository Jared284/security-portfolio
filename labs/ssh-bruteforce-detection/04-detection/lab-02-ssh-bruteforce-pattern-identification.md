# Lab 02 – SSH Brute-Force Pattern Identification

## Objective

Identify patterns of repeated SSH authentication failures that may indicate brute-force attack behavior.

This lab focuses on detecting suspicious authentication patterns by analyzing repeated failed login attempts originating from the same source IP.

---

## Environment

- **Server OS:** Ubuntu Server 24.04 LTS  
- **Service:** OpenSSH Server (sshd)  
- **Client / Attack Source:** Windows host on local network  
- **Log Source:** `/var/log/auth.log`

---

## Procedure

1. Generated multiple failed SSH login attempts from the Windows host  
2. Observed authentication events written to `/var/log/auth.log`  
3. Filtered failed authentication events using command-line tools  
4. Investigated repeated failures originating from the same source IP  

---

## Evidence

Example failed authentication log entry:

```
Failed password for invalid user testuser from 192.168.56.101 port 53412 ssh2
```

Observed pattern:

- Multiple failed login attempts from the same IP address
- Attempts occurring within a short time window
- Repeated probing against non-existent usernames

Summary of observed behavior:

- **Total failed attempts:** 6  
- **Source IP:** `192.168.56.101`

---

## Detection Logic

Brute-force authentication behavior can be detected using the following logic:

1. Extract failed SSH authentication events from the log file
2. Identify the source IP responsible for each event
3. Count repeated failures originating from the same IP address
4. Flag IP addresses generating unusually high failure counts

Repeated authentication failures from a single source IP are significantly more suspicious than isolated login mistakes.

---

## Commands Used

Extract failed authentication attempts:

```
grep "Failed password" /var/log/auth.log
```

Count failed login attempts by source IP:

```
grep "Failed password" /var/log/auth.log | awk '{print $NF}' | sort | uniq -c | sort -nr
```

This command aggregates authentication failures and highlights IP addresses responsible for the most login attempts.

---

## Analysis

Authentication logs alone contain large volumes of raw events.

By aggregating failed login attempts by source IP, patterns begin to emerge that indicate possible brute-force activity.

Key indicators observed in this lab:

- Multiple failed authentication attempts from a single IP
- Repeated probing against invalid usernames
- Clustering of failures within a short time period

These patterns represent early indicators of brute-force authentication attempts.

---

## Outcome

This lab demonstrated how repeated authentication failures can reveal brute-force attack patterns.

By filtering and aggregating authentication logs, analysts can identify suspicious login behavior and prepare detection logic for automated mitigation systems such as Fail2Ban.
