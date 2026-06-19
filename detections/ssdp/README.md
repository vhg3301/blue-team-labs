# Unauthorized SSDP/UPnP Advertisement Detection

> Sigma rule and analyst playbook for detecting UPnP/SSDP traffic crossing trust-zone boundaries. Derived directly from a passive reconnaissance lab in which the same protocol leaked OS version, vendor library, and an internal HTTP endpoint without active scanning.

## TL;DR

SSDP (`udp/1900`) is designed for trusted home LANs and was never meant to cross segmentation boundaries. Its presence in a server VLAN or DMZ indicates either a segmentation failure or an attacker leveraging the protocol for reconnaissance. This detection alerts on SSDP traffic originating from restricted segments, with a documented analyst workflow from alert to containment.

| Field | Value |
|-------|-------|
| Rule file | [`ssdp-unauthorized-segment.yml`](./ssdp-unauthorized-segment.yml) |
| Log source | Firewall (NetFlow / connection logs) |
| MITRE ATT&CK | [T1046](https://attack.mitre.org/techniques/T1046/), [T1018](https://attack.mitre.org/techniques/T1018/) |
| Severity | Medium |
| Status | Experimental — not tested against production telemetry |
| Source lab | [`Red-Team-Labs/passive-recon-ssdp-upnp`](https://github.com/<your-handle>/Red-Team-Labs/tree/main/passive-recon-ssdp-upnp) |

## Why Detect This

SSDP supports UPnP and DLNA — discovery protocols built around the assumption that every device on the LAN should be able to advertise its presence and capabilities to every other device. On a corporate network this assumption fails in two specific ways:

**Information disclosure.** A single SSDP `NOTIFY` packet voluntarily exposes kernel version, vendor library, and HTTP service endpoints to every host that can receive it. In the companion red team lab, one packet revealed `Linux 3.10.104 + RKDLNALib/2.0` and an internal description endpoint at `http://<target>:38389/deviceDescription/MediaServer` — with zero packets sent to the target.

**Architecture violation.** SSDP packets are typically sent with `TTL <= 4` to constrain them to the local broadcast domain. Observing them across a VLAN boundary means either segmentation has failed (misconfiguration) or something is actively relaying them (likely malicious).

## Detection Logic

**Looking for:** SSDP traffic appearing in segments where no UPnP-capable device is authorized — typically server VLANs, DMZ, production workloads.

**Not looking for:** SSDP traffic on user or IoT VLANs where consumer devices legitimately live. Tuning per environment is mandatory.

The rule is published in [`ssdp-unauthorized-segment.yml`](./ssdp-unauthorized-segment.yml). The CIDR ranges in the rule are placeholders — they must be replaced with the actual restricted segments of the target environment before deployment.

## Analyst Workflow

When the rule fires, the following sequence answers the relevant questions in order:

### 1. Identify the source

```bash
nslookup <source-ip>
# or query the asset inventory / CMDB
```

Question: *whose device is this, what is it for, where should it be?*

### 2. Confirm the traffic is real, not a malformed log

Pull the matching PCAP from the network sensor (Zeek, Suricata, or a SPAN-fed Wireshark):

```bash
tshark -r capture.pcap -Y "udp.port==1900 and ip.src==<source-ip>" -V
```

Compare what you see against expected patterns:

| Observation | Likely meaning |
|-------------|----------------|
| `NTS: ssdp:alive` from a device that ships with UPnP enabled | Misplaced asset — move to correct VLAN, disable UPnP if unnecessary |
| `M-SEARCH` from a workstation | Active discovery — possible reconnaissance, investigate the process |
| SSDP from a host with no UPnP role at all (a server, a domain controller) | High suspicion of compromise — escalate |

### 3. Pivot to endpoint

**Linux source:**

```bash
ss -tunap | grep ':1900'
```

**Windows source:**

```powershell
Get-NetUDPEndpoint -LocalPort 1900 | Get-Process -Id { $_.OwningProcess }
```

This identifies the process emitting the SSDP traffic. The same `packet ↔ process` correlation technique is documented in the red team source lab.

### 4. Contain

If the source is unauthorized or shows signs of compromise:

1. Isolate the host at the switch port or via NAC quarantine VLAN
2. Snapshot the host for forensics before any remediation
3. **Audit firewall rules between the source segment and other zones** — SSDP should never have reached your sensor if segmentation were correctly configured. The alert is also a control failure indicator.
4. If multiple hosts are affected, escalate as possible segmentation failure rather than per-host investigation

## Tuning Guidance

- **Per-environment CIDR adjustment is mandatory** — the rule will not work as published
- Consider an inverse rule: alert on SSDP traffic *not* originating from the approved IoT VLAN — same logic, different framing
- Suppress known scanning sources (vulnerability scanners, asset discovery tools) by source IP
- Pair this with a complementary detection for mDNS (`udp/5353`) using equivalent logic

## Caveats

This rule is **experimental**. Specifically:

- It has not been tested against production telemetry
- False positive rates are unknown
- Thresholds and CIDR values are placeholders
- The rule format targets Sigma; conversion to specific SIEMs (Splunk, Sentinel, Elastic) may require adjustment

A detection rule that has not seen production data is a hypothesis. Treat it as such.

## References

- Source lab — [`Red-Team-Labs/passive-recon-ssdp-upnp`](https://github.com/<your-handle>/Red-Team-Labs/tree/main/passive-recon-ssdp-upnp)
- [Sigma rule specification](https://github.com/SigmaHQ/sigma/wiki/Specification)
- [UPnP Device Architecture 2.0](https://openconnectivity.org/developer/specifications/upnp-resources/upnp/)
- [MITRE ATT&CK — T1046 Network Service Discovery](https://attack.mitre.org/techniques/T1046/)
- [MITRE ATT&CK — T1018 Remote System Discovery](https://attack.mitre.org/techniques/T1018/)
