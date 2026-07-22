# Foundations — Environment Setup, jq, tshark & Sniffnet

Goal: go from a clean Ubuntu 22.04 install to a hardened analyst workstation with core log-analysis and packet-inspection tooling operational.

## Sections

1. [Lab Setup](./01-lab-setup/) — VM creation in VMware Workstation
2. [System Hardening](./02-system-hardening/) — patching, core CLI packages, repo prerequisites
3. [jq Fundamentals](./03-jq-fundamentals/) — JSON querying, filtering, and reshaping
4. [jq Real Log Triage](./04-jq-real-log-triage/) — running jq against a realistic multi-alert Wazuh-style export
5. [tshark + jq Pipeline](./05-tshark-jq-pipeline/) — live capture → structured JSON → analyst queries
6. [Sniffnet](./06-sniffnet/) — lightweight visual traffic monitor install

> Note: The Wireshark GUI install/capture/filter lessons (normally part of this stage) were intentionally skipped in favor of going straight to `tshark` — the command-line engine that ships with the same package. This kept the workflow scriptable and loggable rather than screenshot-dependent, and `tshark` covers the same packet-capture capability needed for the pipeline.

## Why this matters for the platform

Every tool here maps to a stage of the eventual pipeline:

| Tool | Later use |
|---|---|
| `jq` | Parses and reshapes JSON from Wazuh, DFIR-IRIS, Cortex, and Shuffle's API responses |
| `tshark` | Automated PCAP-to-JSON conversion, later triggered by Shuffle workflows |
| `Sniffnet` | Quick visual sanity-check of traffic alongside the CLI pipeline |

## Artifacts produced

| File | Section | Purpose |
|---|---|---|
| `sample-alert.json` | 3 | Single nested alert — jq nested-field practice |
| `alerts.json` | 3 | 3-alert array — jq array/select() practice |
| `extracted-iocs.txt` | 3 | High-severity IPs extracted via jq |
| `wazuh-sample.json` | 4 | 5-alert realistic Wazuh export |
| `high-severity-alerts.json` | 4 | Filtered + reshaped triage output (level ≥ 10) |
| `capture-summary.md` | 5 | tshark capture stats and jq query results |

## Checkpoint

A VM snapshot (`Day1-jq-Wireshark-Complete`) was taken at the end of this stage — the safe rollback point before the SIEM Deployment module, which installs Docker and modifies system-level services.
