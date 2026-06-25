--- 
layout: default
title: "Brute Force Attack Investigation"
parent: Phishing Analysis Lab Report
nav_order: 3
---


# Phishing Analysis Lab Report


# 1. Introduction

This report documents my hands-on walkthrough of a phishing investigation scenario on TryHackMe.

The scenario places you in the role of an IT support employee at a fictional company called **SwiftSpend Financial**. Several employees reported receiving suspicious emails, and some had already submitted their credentials before the incident was identified.

The objective of the exercise was to investigate the phishing campaign from the initial email delivery stage through to the attacker's infrastructure and phishing kit.

Skills practiced during this lab include:

* Email analysis
* Attachment analysis
* URL investigation
* Threat intelligence research
* IOC extraction
* Phishing kit analysis
* Basic digital forensics

---

# 2. Environment & Tools

The investigation machine provided a limited but realistic toolset:

* Thunderbird – Email analysis
* CyberChef – URL defanging and encoding operations
* Firefox – Safe browsing and website inspection
* grep – File searching and filtering
* wget – File retrieval
* sha256sum – Hash generation
* VirusTotal – Threat intelligence lookups

No directory enumeration tools such as Gobuster, Dirb, Dirsearch, or FFUF were available, requiring manual investigation techniques.

---

# 3. Investigation Steps

## 3.1 Identifying the Email with a PDF Attachment

The `phish_emails` directory contained five email files.

Rather than opening every email individually, I searched for PDF attachments using:

```bash
grep ".pdf" ./phish_emails/*
```

This quickly revealed that only one email contained a PDF attachment.

After opening the email in Thunderbird, I confirmed the recipient.

### Finding

**William McClean**

---

## 3.2 Identifying the Attacker's Email Address

Reviewing the sender information across all phishing emails showed that they originated from the same address.

### Finding

```text
Accounts.Payable@groupmarketingonline.icu
```

The `.icu` top-level domain is frequently observed in phishing campaigns because it is inexpensive and easy to register.

---

## 3.3 Redirection URL for Zoe Duncan (Defanged)

The email sent to Zoe Duncan contained an HTML attachment rather than a direct link.

After saving and inspecting the HTML file, I identified a redirect to a phishing page hosted on the `kennaroads.buzz` domain.

Using CyberChef, I converted the URL into a safe, defanged format.

### Finding

```text
hxxp[://]kennaroads[.]buzz/data/Update365/office365/40e7baa2f826a57fcf04e5202526f8bd/?email=zoe[.]duncan@swiftspend[.]finance&error
```

The victim's email address is embedded within the URL, likely to increase credibility by pre-populating the phishing form.

---

## 3.4 Locating the Phishing Kit Archive

Reviewing the `/etc/hosts` file showed that the phishing domain was mapped to an internal IP address.

Browsing to the root domain displayed a legitimate-looking WordPress site. However, navigating through exposed directories revealed a downloadable phishing kit archive.

### Finding (Defanged)

```text
hxxp[://]kennaroads[.]buzz/data/Update365[.]zip
```

The exposed archive was likely the result of poor operational security by the attacker.

---

## 3.5 SHA256 Hash of the Archive

After downloading the archive, I generated its SHA256 hash:

```bash
sha256sum Update365.zip
```

### Finding

```text
ba3c15267393419eb08c7b2652b8b6b39b406ef300ae8a18fee4d16b19ac9686
```

---

## 3.6 First Submission to VirusTotal

Using the SHA256 hash, I searched VirusTotal and reviewed the file history.

### Finding

```text
2020-04-08 21:55:50 UTC
```

This indicates that the phishing kit had previously been identified and submitted by other researchers or security professionals.

---

## 3.7 SSL Certificate First Logged

The SSL certificate was no longer available at the time of analysis.

Based on the information provided in the room:

### Finding

```text
2020-06-25
```

---

## 3.8 User Who Submitted Their Password Twice

A file named `log.txt` on the phishing server contained captured victim information.

After downloading the file, I used:

```bash
grep -i Email log.txt | sort | uniq -c
```

This showed one user appearing twice.

Further verification confirmed that the same password was submitted both times.

### Finding

```text
michael.ascot@swiftspend.finance
```

This behavior is common when phishing pages intentionally display an error message to encourage victims to re-enter their credentials.

---

## 3.9 Attacker's Credential Collection Email

After extracting the phishing kit, I analyzed the PHP files responsible for processing submitted credentials.

The key file was:

```text
Update365/office365/Validation/submit.php
```

The script forwarded stolen credentials to the following email address.

### Finding

```text
m3npat@yandex.com
```

---

## 3.10 Secondary Gmail Address Found in the Kit

Searching the extracted files for Gmail addresses revealed another email account referenced throughout the kit.

```bash
grep -r "gmail.com" ./Update365/
```

### Finding

```text
jamestanner2299@gmail.com
```

This may belong to the original phishing kit author or a previous operator.

---

## 3.11 Hidden Flag

Without directory enumeration tools, I manually tested common file names.

Accessing:

```text
/data/Update365/office365/flag.txt
```

revealed a hidden file.

The contents were reversed and Base64 encoded.

### Finding

```text
THM{pL4y_w1Th_tH3_URL}
```

---

# 4. Summary of Findings

| #  | Question                          | Answer                                                                                        |
| -- | --------------------------------- | --------------------------------------------------------------------------------------------- |
| 1  | Who received the PDF attachment?  | William McClean                                                                               |
| 2  | Attacker's sending address        | [Accounts.Payable@groupmarketingonline.icu](mailto:Accounts.Payable@groupmarketingonline.icu) |
| 3  | Redirect URL for Zoe Duncan       | hxxp[://]kennaroads[.]buzz/data/Update365/office365/...                                       |
| 4  | Phishing kit archive URL          | hxxp[://]kennaroads[.]buzz/data/Update365[.]zip                                               |
| 5  | SHA256 Hash                       | ba3c15267393419eb08c7b2652b8b6b39b406ef300ae8a18fee4d16b19ac9686                              |
| 6  | First VirusTotal submission       | 2020-04-08 21:55:50 UTC                                                                       |
| 7  | SSL certificate first logged      | 2020-06-25                                                                                    |
| 8  | User who submitted password twice | [michael.ascot@swiftspend.finance](mailto:michael.ascot@swiftspend.finance)                   |
| 9  | Credential collection email       | [m3npat@yandex.com](mailto:m3npat@yandex.com)                                                 |
| 10 | Secondary Gmail address           | [jamestanner2299@gmail.com](mailto:jamestanner2299@gmail.com)                                 |
| 11 | Hidden flag                       | THM{pL4y_w1Th_tH3_URL}                                                                        |

---

# 5. Key Takeaways

Several important lessons stood out during this investigation:

* HTML attachments can be used to hide phishing URLs from traditional email filtering systems.
* Misconfigured web servers with directory listing enabled can expose phishing kits and attacker resources.
* Storing captured credentials in plaintext creates additional risks for attackers if their infrastructure is discovered.
* Purchased or reused phishing kits often contain leftover information from previous operators.
* Threat intelligence platforms such as VirusTotal can provide useful historical context and help identify reused infrastructure.

---

# 6. Conclusion

This lab provided practical experience investigating a phishing campaign from start to finish.

The exercise covered email analysis, phishing infrastructure discovery, threat intelligence gathering, phishing kit analysis, and IOC extraction. It also demonstrated how attackers collect credentials and how small operational mistakes can expose valuable intelligence to defenders.

Overall, this room was a useful introduction to phishing investigations and helped reinforce many of the techniques commonly used by SOC analysts and incident responders.
