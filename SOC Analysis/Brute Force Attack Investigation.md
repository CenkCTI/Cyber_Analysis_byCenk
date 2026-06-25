--- 
layout: default
title: "Brute Force Attack Investigation"
parent: SOC Analysis / TTP Detection
nav_order: 1
---



# Brute Force Attack Investigation

## Summary

During the investigation, I identified a brute force attack targeting the host **tryhackme-2404**. The attack originated from IP address **10.10.242.248**.

The attacker attempted to log in to a user account multiple times and eventually gained access to the account **john.smith**. After obtaining access, the attacker escalated privileges to the **root** account and created a new user account to maintain access to the system.

Based on the collected evidence, the activity was determined to be malicious.

---

## Alert Details

| Field          | Value                          |
| -------------- | ------------------------------ |
| Alert Name     | Brute Force Activity Detection |
| Host           | tryhackme-2404                 |
| Source IP      | 10.10.242.248                  |
| Detection Time | 17 Sep 2025 09:00:21           |

---

## Investigation Process

### 1. Failed Login Attempts

I first reviewed authentication logs related to the source IP address.

Query used:

```spl
src_ip=10.10.242.248 action=failure
| stats count by user
```

The results showed that the account **john.smith** received approximately **500 failed login attempts**.

| User       | Failed Attempts |
| ---------- | --------------- |
| john.smith | 500             |

This behavior is consistent with a brute force attack.

---

### 2. Attack Timeline

To understand how long the attack lasted, I checked the timestamps of the failed login events.

Query used:

```spl
src_ip=10.10.242.248 action=failure
| table _time,user
```

Observations:

* First failed login: 09:00:21
* Last failed login: 09:05:13

The brute force activity continued for roughly **5 minutes**.

---

### 3. Successful Authentication

After the failed attempts, I searched for successful login events from the same activity.

I found that the attacker successfully authenticated as:

* john.smith

Time of successful login:

* 09:06:01

This indicates that the brute force attack succeeded in compromising the account.

---

### 4. Privilege Escalation Activity

While reviewing subsequent events, I identified the following log entry:

```text
session opened for user root by john.smith
```

This suggests that the attacker obtained root-level privileges after compromising the user account.

---

### 5. Persistence

Further analysis revealed the creation of a new local account:

```text
useradd system-utm
```

Created account:

* system-utm

Creating a new account is a common persistence technique that allows an attacker to regain access later.

---

## Findings

| Finding               | Result             |
| --------------------- | ------------------ |
| Brute Force Attack    | Confirmed          |
| Compromised Account   | john.smith         |
| Failed Login Attempts | 500                |
| Privilege Escalation  | root               |
| Persistence Mechanism | system-utm account |

---

## MITRE ATT&CK Techniques

* T1110 – Brute Force
* T1078 – Valid Accounts
* T1068 – Privilege Escalation
* T1136 – Create Account

---

## Conclusion

This investigation confirmed a successful brute force attack against the account **john.smith**. After gaining access, the attacker escalated privileges to **root** and created a new account (**system-utm**) to maintain persistence on the system.

If this were a real environment, recommended actions would include resetting compromised credentials, removing unauthorized accounts, reviewing privilege escalation paths, and monitoring the affected host for additional malicious activity.
