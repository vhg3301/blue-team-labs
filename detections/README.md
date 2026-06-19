# Blue-Team-Labs

Detection engineering, threat hunting workflows, and SOC analyst playbooks. Focused on turning attacker behavior into defensible, observable signal.

## Scope

This repository covers the defensive side of my cybersecurity practice:

- **Detection rules** — Sigma, Wazuh, and KQL signatures derived from observed or replicated attacker behavior
- **Threat hunting** — hypothesis-driven hunts with documented methodology
- **SOC workflows** — analyst-facing playbooks for alert triage and investigation

Sister repositories:

- **`Red-Team-Labs`** — the offensive side. Many detections here are derived directly from labs there.
- **`Security-Notes`** — protocol and concept references the detections rely on
- **`Scripts`** — tooling

## Contents

### Detections

| Rule | Targets | Source | Status |
|------|---------|--------|--------|
| [`detections/ssdp-anomaly-detection.md`](./detections/ssdp-anomaly-detection.md) | Unauthorized UPnP/SSDP traffic crossing trust boundaries (T1046, T1018) | Derived from [`Red-Team-Labs/passive-recon-ssdp-upnp`](https://github.com/<your-handle>/Red-Team-Labs/tree/main/passive-recon-ssdp-upnp) | Experimental |

### Planned

- Wazuh custom rules for the home lab SIEM project
- Threat hunting playbook for Active Directory enumeration (Kerberoasting, BloodHound-style queries)
- SOC triage workflow for suspected lateral movement

## How These Documents Are Structured

Each detection follows the same outline:

1. **Why detect this** — the underlying offensive technique and business risk
2. **Detection logic** — what signal we are looking for, and why it indicates the technique
3. **Rule** — Sigma/Wazuh/KQL with placeholders for environment-specific values
4. **Analyst workflow** — step-by-step triage from alert to containment
5. **Tuning guidance** — known false positives and how to reduce them
6. **MITRE ATT&CK mapping**
7. **References**

This is the format I would want as a SOC analyst receiving an alert: enough context to understand *why* the rule fired, not just *what* fired it.

## A Note on Stance

The detections here are **experimental** unless explicitly marked otherwise. They reflect detection logic I would propose, not validated production rules. Specifically:

- Threshold and CIDR values are placeholders and must be tuned per environment
- False positive rates have not been measured against real telemetry
- The rules have not been tested at scale on production SIEMs

This is deliberate honesty, not a disclaimer. A detection rule that has not seen production data is a hypothesis. Treating it as anything else is how SOCs end up with alert fatigue and broken trust between detection engineering and L1 analysts.

## Red ↔ Blue Loop

Where possible, each detection here links to the offensive lab that motivated it. The intent is to demonstrate a working understanding of both sides of the same technique — the attacker's mechanics and the defender's signal — rather than presenting either in isolation.
