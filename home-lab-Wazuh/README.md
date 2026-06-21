# Wazuh Home Lab

A hands-on Blue Team home lab built to simulate real-world Security Operations Center (SOC) workflows using Wazuh SIEM.

This repository documents security monitoring, threat detection, log analysis, incident triage, and detection engineering exercises performed in a controlled environment. The goal is to develop practical SOC Analyst skills through realistic attack simulations and incident investigations.

---

## Objectives

* Build and maintain a functional SIEM environment
* Practice security monitoring and alert triage
* Investigate real attack scenarios in a lab environment
* Improve log analysis and incident response skills
* Apply MITRE ATT&CK mapping to detected activities
* Develop SOC Level 1 analyst methodologies

---

## Lab Environment

| Component        | Description                                        |
| ---------------- | -------------------------------------------------- |
| SIEM             | Wazuh Manager & Dashboard                          |
| Agent            | Wazuh Agent                                        |
| Target Systems   | Ubuntu Server                                      |
| Attacker Systems | Linux Mint                                         |
| Log Sources      | SSH, Syslog, Journalctl, Linux Authentication Logs |

---

## Skills Practiced

* Security Monitoring
* Log Analysis
* Incident Triage
* IOC Identification
* Threat Detection
* MITRE ATT&CK Mapping
* Linux Security Analysis
* Wazuh Rule Investigation
* Security Incident Documentation
* SOC Reporting

---

## Case Studies

### SSH Brute Force Detection

**Description:** Detection and investigation of an SSH brute force attack using Wazuh SIEM.

**Topics Covered:**

* SSH authentication log analysis
* Wazuh alert investigation
* MITRE ATT&CK T1110 (Brute Force)
* IOC identification
* Incident classification
* Security recommendations

**Report:**

```text
Detecting-SSH-attack.md
```

---

## Repository Structure

```text
home-lab-Wazuh/
│
├── README.md
│
├── Detecting-SSH-attack.md
│
└── images/
    ├── logs_of_brute_force_simulation.png
    └── siem_logs_brute_force.png
```

---

## Methodology

Each case study follows a structured SOC workflow:

1. Attack Simulation
2. Log Collection
3. SIEM Detection
4. Alert Triage
5. IOC Identification
6. MITRE ATT&CK Mapping
7. Incident Classification
8. Reporting and Documentation
9. Mitigation Recommendations

---

## Future Scenarios

Planned investigations include:

* Web Attack Detection
* Privilege Escalation Detection
* Suspicious Process Execution
* Malware Activity Simulation
* Persistence Mechanisms
* Lateral Movement Detection
* Account Compromise Scenarios
* File Integrity Monitoring Alerts

---

## Disclaimer

All activities documented in this repository were performed in a controlled laboratory environment for educational and defensive security purposes only.
