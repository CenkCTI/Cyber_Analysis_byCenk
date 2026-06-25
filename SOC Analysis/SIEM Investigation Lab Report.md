--- 
layout: default
title: "SIEM Investigation Lab Report"
parent: SOC Analysis / TTP Detection
nav_order: 1
---


# SIEM Investigation Lab Report

## Overview

This lab was completed as part of a SIEM training exercise in TryHackMe. The objective was to investigate security events using Splunk and analyze logs from different sources, including Windows endpoints, Linux servers, and web applications.

During the lab, I practiced searching and correlating events, identifying suspicious activities, investigating indicators of compromise, and building a timeline of attacker actions.

---

## Skills Practiced

* Splunk log analysis
* SIEM investigation workflow
* Event correlation
* Windows event log analysis
* Linux authentication log analysis
* Web server log analysis
* Threat hunting techniques
* Incident timeline creation
* Basic attacker behavior identification

---

# Part 1 – Windows Log Investigation

### Scenario

An alert was generated for a suspicious network connection on host **WIN-105** using destination port **5678**. The objective was to determine whether the activity was malicious and identify the process responsible for the connection.

### Investigation Process

I reviewed Sysmon network connection events and filtered logs related to the suspicious port. By examining process information and associated hashes, I was able to trace the activity back to a specific executable.

### Findings

* Identified a suspicious outbound connection from WIN-105.
* Determined the remote IP address involved in the communication.
* Identified the process responsible for establishing the connection.
* Retrieved the MD5 hash of the executable for further analysis.
* Discovered that a scheduled task had been created on the system, indicating a persistence mechanism.

### Conclusion

The investigation revealed behavior commonly associated with malware activity, including suspicious network communication and persistence through a scheduled task.

---

# Part 2 – Linux Log Investigation

### Scenario

An alert indicated possible persistence activity on an Ubuntu server through the creation of a new user account named **remote-ssh**.

### Investigation Process

I reviewed authentication and system logs to reconstruct the sequence of events leading to the account creation.

### Findings

* Identified the timestamp when the new account was created.
* Determined which user successfully escalated privileges to root.
* Traced the successful login back to its source IP address.
* Counted failed authentication attempts preceding the successful login.
* Identified a persistence mechanism configured through a scheduled task.

### Attack Timeline

1. Multiple failed login attempts were observed.
2. A successful login occurred from a remote IP address.
3. The attacker escalated privileges using sudo.
4. A new user account was created.
5. Persistence was established through scheduled task configuration.

### Conclusion

The logs showed a clear sequence of unauthorized access, privilege escalation, and persistence establishment. The activity was consistent with a successful compromise of the system.

---

# Part 3 – Web Application Log Investigation

### Scenario

An alert reported unusually high activity against a web server. The goal was to determine the source and nature of the activity.

### Investigation Process

I analyzed HTTP request logs and reviewed request volumes, source IP addresses, request methods, and user-agent strings.

### Findings

* Identified the most frequently requested URI.
* Determined the source IP responsible for the majority of requests.
* Observed hundreds of authentication attempts against the WordPress login page.
* Detected the use of automated tooling through the user-agent string.
* Identified the attacker tool as WPScan.

### Assessment

The activity was classified as a brute-force attack targeting WordPress authentication.

### Indicators

* High volume of login requests
* Automated scanning behavior
* Repeated POST requests to wp-login.php
* Use of offensive security tooling

### Conclusion

The web logs showed clear evidence of automated reconnaissance and brute-force activity targeting a WordPress application.

---

# Key Learning Outcomes

Through this lab I gained practical experience with:

* Investigating alerts in a SIEM environment
* Searching and filtering logs using Splunk
* Correlating events across multiple log sources
* Analyzing Windows, Linux, and web application logs
* Identifying attacker techniques such as persistence, privilege escalation, and brute-force attacks
* Building investigation timelines and documenting findings

---

## Tools Used

* Splunk
* Sysmon Logs
* Windows Event Logs
* Linux Authentication Logs
* Web Server Logs
* TryHackMe SIEM Training Environment

---

## Final Notes

This lab simulated the daily activities of a SOC Level 1 Analyst. The exercises focused on alert investigation, log analysis, event correlation, and identifying suspicious behavior across different environments. The experience helped reinforce fundamental SIEM investigation techniques and incident analysis workflows.
