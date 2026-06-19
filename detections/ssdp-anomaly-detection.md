# Detecting Unauthorized UPnP/SSDP Advertisements

> Sigma detection rule and analyst guidance for identifying SSDP NOTIFY traffic that crosses VLAN or trust-zone boundaries — derived from a passive reconnaissance lab where this same protocol leaked OS version, vendor stack, and exposed endpoints.

## Why Detect This

SSDP (`udp/1900`) is the discovery protocol behind UPnP and DLNA. It was designed for trusted home LANs where any device should be able to advertise its presence and capabilities. On a corporate network this is a problem for two reasons:

1. **Information disclosure** — SSDP `NOTIFY` packets voluntarily expose kernel version, vendor libraries, and HTTP endpoints to every host that can receive them. A companion lab in this portfolio demonstrated how a single packet revealed `Linux 3.10.104 + RKDLNALib/2.0` and an internal description endpoint, with zero packets sent to the target. See [`Red-Team-Labs/passive-recon-ssdp-upnp/`](../../Red-Team-Labs/passive-recon-ssdp-upnp/).
2. **Network architecture violation** — SSDP traffic with `TTL <= 4` is intentionally constrained to the local broadcast domain. Observing it crossing a VLAN boundary means either a misconfiguration (segmentation has failed) or an active relay (likely malicious).

## Detection Logic

**What we are looking for:** SSDP traffic appearing in a segment where no UPnP-capable device is authorized to exist — typically server VLANs, DMZ, or production workload subnets.

**What we are *not* looking for:** SSDP traffic on user/IoT VLANs where consumer devices live. Tuning by network segment is mandatory.

## Sigma Rule

```yaml
title: Unauthorized SSDP/UPnP Advertisement in Restricted Network Segment
id: 9b3e4f2a-7c81-4d9e-b6a1-3f5d8e0c2a47
status: experimental
description: |
  Detects SSDP NOTIFY or M-SEARCH traffic (UDP/1900) originating from
  or destined to network segments where UPnP devices are not authorized.
  Legitimate UPnP traffic is expected on user/IoT VLANs; its appearance
  in server, DMZ, or production subnets indicates either segmentation
  failure or an active relay attempt.
author: Vitor
date: 2026/06/19
references:
  - https://attack.mitre.org/techniques/T1046/
  - https://attack.mitre.org/techniques/T1018/
  - https://openconnectivity.org/developer/specifications/upnp-resources/upnp/
tags:
  - attack.discovery
  - attack.t1046
  - attack.t1018
logsource:
  category: firewall
detection:
  selection_protocol:
    dst_port: 1900
    protocol: udp
  selection_segment:
    src_ip|cidr:
      - '10.10.0.0/16'    # SERVER_VLAN — adjust to environment
      - '10.20.0.0/16'    # DMZ — adjust to environment
  condition: selection_protocol and selection_segment
falsepositives:
  - Legitimate UPnP devices intentionally placed in restricted segments (rare; should be documented)
  - Network discovery tooling run from administrative hosts
  - Misconfigured IoT devices that should be moved to the IoT VLAN
level: medium
```

## Analyst Workflow on Alert

When this rule fires, the following sequence answers the relevant questions in order:

### 1. Identify the source host

```bash
# From a SOC workstation with appropriate access
nslookup <source-ip>
# or query the asset inventory / CMDB
```

The question is *whose device is this?* — owner, business function, expected location.

### 2. Confirm the traffic is real, not a malformed log

Pull the matching PCAP from the network sensor (Zeek, Suricata, or a SPAN-fed Wireshark) and inspect the SSDP payload:

```bash
tshark -r capture.pcap -Y "udp.port==1900 and ip.src==<source-ip>" -V
```

Expected legitimate output: a NOTIFY with `NTS: ssdp:alive` from a known device class.
Suspicious output: M-SEARCH probes (active discovery) or NOTIFY from a host that has no business being a UPnP device (a server, a workstation, a printer).

### 3. Determine intent

| Observation | Likely meaning |
|------------|----------------|
| `NTS: ssdp:alive` from a device that ships with UPnP enabled | Misconfigured / misplaced asset. Move to correct VLAN, disable UPnP if not needed. |
| `M-SEARCH` from a workstation | Active discovery — possible reconnaissance. Investigate the process generating the traffic. |
| SSDP from a host with no UPnP role at all | High suspicion. Could indicate compromise where attacker is using SSDP as a discovery channel. |

### 4. Pivot to endpoint

On the source host (if Linux/Unix):

```bash
ss -tunap | grep ':1900'
```

On Windows, equivalent:

```powershell
Get-NetUDPEndpoint -LocalPort 1900 | Get-Process -Id { $_.OwningProcess }
```

This identifies the process emitting SSDP traffic — the same `packet ↔ process` correlation technique covered in the companion red team lab.

## Containment

If the source is unauthorized or compromised:

1. Isolate the host at the switch port (or via NAC quarantine VLAN)
2. Snapshot the host for forensic analysis before any remediation
3. Audit firewall rules between the source segment and other zones — SSDP should never have reached your sensor if segmentation were correctly configured
4. If multiple hosts are affected, escalate as a possible segmentation failure rather than per-host investigation

## Tuning Guidance

- **Per-environment IP ranges are mandatory** — the rule as published uses placeholder CIDRs and will not work without adjustment
- Consider an inverse rule: alert on SSDP traffic *not* originating from the approved IoT VLAN
- Suppress known scanning sources (vulnerability scanners, asset discovery tools) by source IP
- Pair this rule with a complementary detection for mDNS (`udp/5353`) using the same logic

## MITRE ATT&CK Coverage

| Technique | ID |
|-----------|-----|
| Network Service Discovery | [T1046](https://attack.mitre.org/techniques/T1046/) |
| Remote System Discovery | [T1018](https://attack.mitre.org/techniques/T1018/) |
| Network Sniffing (related, on the offensive side) | [T1040](https://attack.mitre.org/techniques/T1040/) |

## References

- Companion red team lab: [`Red-Team-Labs/passive-recon-ssdp-upnp/`](../../Red-Team-Labs/passive-recon-ssdp-upnp/)
- [Sigma rule specification](https://github.com/SigmaHQ/sigma/wiki/Specification)
- [UPnP Device Architecture 2.0](https://openconnectivity.org/developer/specifications/upnp-resources/upnp/)
- [RFC 6970](https://datatracker.ietf.org/doc/html/rfc6970) — UPnP IGD attack surface considerations
