# Phishing Email Analysis — Netflix Account Recovery Scam

## Overview

This repository documents the analysis of a phishing email that impersonates Netflix and attempts to trick recipients into clicking a malicious account recovery link.

The objective of this investigation is to identify social engineering techniques, analyze indicators of compromise (IOCs), assess the threat from a Blue Team perspective, and document defensive recommendations.

This project simulates a real-world email threat analysis workflow commonly performed by SOC Analysts and Security Analysts.

---

## Scenario

The analyzed email claims that suspicious activity was detected on a Netflix account and urges the recipient to recover access immediately.

To increase credibility, the attacker impersonates a security operations team ("GSOC"), includes a location and IP address, and creates a false sense of urgency.

The ultimate goal appears to be credential theft through a phishing website.

---

## Investigation Objectives

* Analyze phishing indicators
* Identify social engineering techniques
* Document Indicators of Compromise (IOCs)
* Assess risk level
* Provide defensive recommendations
* Demonstrate Blue Team email analysis methodology

---

## Evidence Collected

### Inbox View

The first screenshot captures the email as received by the victim, including sender information and subject line.

![Inbox View](images/phishing_email_inbox.png)

---

### Email Content

The second screenshot contains the complete phishing email body used during the investigation.

![Email Content](images/phishing_email_content.png)

---

## Key Findings

### Social Engineering Techniques Identified

| Technique           | Purpose                                             |
| ------------------- | --------------------------------------------------- |
| Authority           | Impersonates a security operations center           |
| Fear                | Claims unauthorized access and potential compromise |
| Urgency             | Demands action within a short timeframe             |
| Forced Action       | Encourages immediate link clicking                  |
| Brand Impersonation | Uses Netflix branding to gain trust                 |

---

### Indicators of Compromise (IOCs)

| Type                  | Indicator                     |
| --------------------- | ----------------------------- |
| Sender Domain         | Gmail-based sender            |
| Brand Abuse           | Netflix impersonation         |
| Attack Method         | Credential harvesting         |
| Social Engineering    | Fear and urgency manipulation |
| User Action Requested | Click recovery link           |

---

## Red Flags Identified

### Suspicious Sender Domain

The email was not sent from an official Netflix domain.

Legitimate security communications are typically sent from verified corporate domains rather than free email providers.

---

### Credential Harvesting Pattern

The email attempts to drive the user toward an account recovery link.

This behavior is commonly associated with phishing campaigns designed to steal credentials.

---

### Artificial Urgency

The message pressures the recipient to act within a limited time window.

Creating urgency is a classic social engineering tactic used to bypass rational decision-making.

---

### Brand and Tone Inconsistencies

The email references both a "Global Security Operations Center (GSOC)" and "Netflix Help Center," creating inconsistencies that legitimate communications rarely contain.

---

## Risk Assessment

| Category                         | Assessment |
| -------------------------------- | ---------- |
| Writing Quality                  | High       |
| Technical Legitimacy             | Low        |
| Social Engineering Effectiveness | High       |
| Overall Risk                     | High       |

---

## Defensive Recommendations

### For End Users

* Never click account recovery links directly from emails
* Access services through official websites or applications
* Verify sender domains before taking action
* Enable Multi-Factor Authentication (MFA)
* Use a password manager with unique passwords

### For Organizations

* Implement SPF, DKIM, and DMARC
* Conduct phishing awareness training
* Deploy email filtering solutions
* Add external sender warning banners
* Encourage user reporting of suspicious emails

---

## Skills Demonstrated

* Phishing Email Analysis
* Social Engineering Detection
* IOC Identification
* Threat Assessment
* Security Awareness Evaluation
* Incident Documentation
* Blue Team Reporting

---

## Key Lesson

Modern phishing campaigns often contain well-written and convincing content, sometimes generated or enhanced by AI.

Detection should focus on:

* Sender legitimacy
* Link destinations
* Requested actions
* Psychological manipulation techniques

rather than relying solely on grammar or spelling mistakes.

---

## Conclusion

This phishing email demonstrates how attackers combine authority, urgency, and brand impersonation to increase the likelihood of user interaction.

Although the message appears convincing at first glance, several indicators reveal its malicious nature, including the sender domain mismatch, recovery-link strategy, and inconsistent branding.

The analysis reinforces the importance of user awareness and proper email verification practices as critical defenses against phishing attacks.
