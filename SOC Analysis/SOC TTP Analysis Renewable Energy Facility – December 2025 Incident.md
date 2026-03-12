--- 
layout: default
title: "rapor1"
parent: SOC Analysis / TTP Detection -Tactical Level
nav_order: 1
---
**SOC TTP Analysis: Renewable Energy Facility – December 2025 Incident**

## 1. Executive Summary
 

The incident was **detected in December 2025**, with activity **dating back to March 2025**. The **initial access vector could not be confirmed**.

**Initial indicators of compromise** were detected on the **Fortigate firewall device**. Following initial access, the attacker performed **unauthorized access, lateral movement, and further system compromise** within the environment.

The incident resulted in **operational disruption**, including communication loss between IT/OT systems and industrial control devices. However, **electricity generation was not affected**.


---

# 2. Target Environment Overview


### 2.1 Grid Connection Point (GCP) Architecture

Within the industrial automation domain, the key GCP components rele‑
vant to the described attack include:
• Remote Terminal Unit (RTU): responsible for telecontrol functions and
supervision of the substation’s operation.
• Local HMI: used to visualize the operational status of the substation
based on data provided by the RTU.
• Protection relays: responsible for protection functions, including fault de‑
tection and isolation.
• Serial device servers: used to connect devices utilizing RS232 or RS485
interfaces and to provide IP‑to‑serial connectivity, enabling communica‑
tion with the DSO where a serial interface is required.
• Primary and backup communication links (a cellular router): used to con‑
nect to the distribution system operator’s SCADA system via DNP3.0 or
IEC 101 protocols.
• Integrated VPN concentrator and firewall: used to provide remote service
access, network segmentation, and, where applicable, connectivity be‑
tween the renewable energy facility’s systems and the DSO.



---

### 2.2 Network Exposure

The GCP infrastructure connects to external networks through a Fortigate firewall device. The Fortigate firewall provides VPN access for remote managemet of the GCP network. The facility enviroment included HMI systems (Windows 10), routers and remote management interfaces used for communication with SCADA  systems and the Distribution System Operator.
 
 
 ---

# 3. Initial Access Vector

Investigation findings suggest that the inital access point was likely the Fortigate firewall device. 
Evidence of connections originating from the tor network. combined with the weak administrative credentials and the lack of multi-factorauthentication, indicates that the internet-exposed management interface of the firewall may have served as the entry point of the network.


### 3.1 Internet-Exposed Devices

Evidence shows that the Fortigate firewall device was exposed to the public internet to allow remote administrative access. The firewall provides VPN connectivity, enabling access to subnets in other facilities.
### 3.2 Credential Attacks

Evidence suggests that weaknes of the compromised devices were weak administrative passwords, lack of multi-factor authentication, default credential on some devices such as  Mikronika HMI Computers.

---

# 4. Post-Exploitation Activities


Evidence suggests that the attacker gained access to Windows 10 HMI systems (such as Mikronika HMI computers) using RDP and local administrator privileges, taking advantage of weak or default credentials.

### 4.2 System Configuration Manipulation

Investigation findings show that the attacker executed PowerShell commands to enable SMB administrative shares, restart the SMB service, and create a new firewall rule named 'Microsoft Update' allowing TCP port 445. The manipulated configurations made the machine remotely accessible via SMB, facilitating lateral movement, remote payload deployment, and internal network propagation.




---

### 4.3 Lateral Movement Preparation

Investigation findings indicate that in cases where HMI systems used different credential for the local administrator account, password-breaking attemps were unsuccessful. As a result, those HMI systems were not compromised or damaged.

---

# 5. ICS / OT Device Manipulation

### 5.1 RTU Controller Sabotage

Analysis revealed that the attacker modified the firmware of Hitachi RTU 560 controllers. The malicious firmware altered the first 240 bytes to 0xF values, causing the device to return empty responses and enter a reboot loop. As a result, the affected RTU controllers became non-operational. The firmware identified itself as version 13.5.3.0, a version not used in any of the affected facilities, suggesting that the attacker likely obtained the firmware from an external environment.

---

### 5.2 Serial Device Server Disruption

Affected facilities were using Moxa NPort 6xxx serial device servers. Access to the web interface was enabled on these devices and the login credentials remained set to their default values. The attacker used these credentials to log in, perform a factory reset, and change the device password. The attacker then configured the device IP address to 127.0.0.1. As a result, the devices appeared to remain operational but became unreachable from the network, causing a loss of communication between facilities.

---

# 6. Malware Deployment

Evidence suggests that after enabling SMB access on TCP port 445, the attacker uploaded a malware file named `C:\Source.exe` to the compromised HMI system. The same malware was later observed in attacks against Combined Heat and Power (CHP) plants.

---

# 7. Attacker Tactics, Techniques and Procedures (TTP)


| Tactic           | Technique                 | ID    |
| ---------------- | ------------------------- | ----- |
| Initial Access   | External Remote Services  | T1133 |
| Initial Access   | Valid Accounts            | T1078 |
| Execution        | Command Execution         | T1569 |
| Lateral Movement | Remote Services           | T1021 |
| Discovery        | Network Service Discovery | T1046 |
| Impact           | System Shutdown/Reboot    | T1529 |
|                  |                           |       |

---

# 8. Operational Impact

The attack caused **loss of SCADA communication** with the affected facilities and **disrupted remote management capabilities** for operators. Some devices experienced **permanent damage** and required **manual intervention** to restore functionality, increasing **recovery complexity**. Despite these operational disruptions, **electricity generation was not affected**, and power supply to the grid continued without interruption.

---

# 9. Security Control Failures

Evidence shows that devices in the facilities, such as Fortigate firewalls and RTUs, used weak or default login credentials. The absence of multi-factor authentication allowed attackers to remain undetected. Centralized logging was not implemented, hindering threat detection, investigation, and incident response. Firmware updates were not regularly applied, as updating the systems could potentially disrupt facility operations, creating a security gap. It is likely that personnel responsible for deploying and operating IT/OT systems had limited security awareness and training, resulting in low-security configurations during system setup.

---

# 10. Detection Opportunities for SOC

Suspicious connections originating from the Tor network and repeated failed login attempts could have been detected by behavior-based SIEM or SOC monitoring. If multi-factor authentication (MFA) had been enabled, unauthorized logins could have been immediately noticed by personnel. Restricting user privileges according to operational requirements and limiting facility personnel accounts from performing actions such as firmware updates or firewall rule modifications would have deterred the attacker.

---

# 11. Key Lessons for OT Security

Establishing policies that explain security measures and risks will help personnel and IT/OT teams in the facilities. Organizing seminars to define and raise awareness among these personnel regarding their responsibilities in cybersecurity is also beneficial. Behavioral-based SIEMs are efficient in identifying anomalies in this incident. Regularly updating devices, or configuring them in the most secure way if regular updates are not possible, increases device security. Passwords should not be left as default; stronger passwords should be used.
