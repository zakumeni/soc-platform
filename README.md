# Blue Team SOC Platform Lab

A self-built Security Operations Center lab, documented module by module — from a clean Ubuntu install to a working threat detection and incident-response pipeline.

This repo tracks a progressive build: hardened base OS → JSON/log tooling → packet analysis → (upcoming) Wazuh SIEM, TheHive case management, Cortex enrichment, and Shuffle SOAR automation.

## Why this exists

Most "I did a cybersecurity course" repos are a folder of screenshots. This one isn't — every command here was actually run against a real VM, every output is unedited terminal output (including the messy parts), and every artifact (JSON files, IOC lists, capture summaries) is the real thing the tooling produced.

## Lab Environment

- **Hypervisor:** VMware Workstation (not ESXi — chosen to match a standard hosted-hypervisor setup for portability; the same build has also been tested against an ESXi-based homelab)
- **Guest OS:** Ubuntu 22.04.5 LTS
- **VM name:** `BlueTeam-Workstation`
- **Networking:** NAT
- **Spec:** 4 vCPU / 8GB RAM / 100GB disk (thin-provisioned)

## Roadmap

| Module | Focus | Status |
|---|---|---|
| [Foundations](./foundations/) | Environment hardening, jq, tshark, Sniffnet | ✅ Complete |
| [SIEM Deployment](./siem-deployment/) | Docker + Wazuh SIEM deployment | 🔜 Planned |
| [Case Management](./case-management/) | Wazuh rule tuning, TheHive case management | 🔜 Planned |
| [SOAR Automation](./soar-automation/) | Cortex enrichment + Shuffle SOAR automation | 🔜 Planned |

## Foundations Summary

Environment setup, JSON/log analysis with `jq`, and packet capture tooling. Full writeup: [foundations/README.md](./foundations/README.md)

- Hardened Ubuntu base (patched, core CLI tooling installed)
- `jq` fundamentals through real Wazuh-style alert triage (filtering, reshaping, grouping, incident summaries)
- Live traffic capture via `tshark`, exported to structured JSON, queried for top talkers / DNS / destination ports
- `Sniffnet` installed as a lightweight visual traffic monitor

## License

MIT — see individual sections for tool-specific licensing (Wireshark, Wazuh, TheHive, Cortex, and Shuffle each have their own).
