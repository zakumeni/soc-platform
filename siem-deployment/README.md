# SIEM Deployment — Docker + Wazuh

Goal: stand up a full Wazuh SIEM stack via Docker Compose, enroll a real Windows endpoint, configure File Integrity Monitoring, and build the detection infrastructure needed to catch process-based attacks (reverse shells, suspicious process trees).

## Environment

- **SIEM host:** the same Ubuntu 22.04 `BlueTeam-Workstation` VM from [Foundations](../foundations/)
- **Monitored endpoint:** a real Windows 10 host (not a disposable VM) — see the note on scope below
- **Wazuh version:** 4.9.2, deployed via the official `wazuh-docker` repo, single-node profile

## Sections

1. [Docker Installation](./01-docker-install/) — Docker Engine + Compose from the official repo
2. [Wazuh Stack Deployment](./02-wazuh-stack-deployment/) — Manager, Indexer, Dashboard via Docker Compose
3. [Windows Agent Enrollment](./03-windows-agent-enrollment/) — enrolling a real Windows endpoint
4. [FIM: Registry & Files](./04-fim-registry-files/) — real-time file integrity monitoring
5. [Reverse Shell Detection](./05-reverse-shell-detection/) — audit policy, Sysmon, detection infrastructure

## A note on scope: real host vs. disposable VM

Sections 1-4 are pure infrastructure/telemetry - safe to run against any machine. Section 5 onward, the curriculum calls for running actual offensive payloads (msfvenom-generated reverse shells, privilege escalation techniques) against the monitored endpoint to generate real detections.

Since the monitored endpoint here is a real daily-use Windows machine rather than a disposable VM, **payload-triggered sections build and verify the detection infrastructure live, but document the actual attack techniques conceptually rather than executing them.** Every log source, audit policy, and detection pipeline is real and tested - the specific offensive payloads are described, not run. This is called out explicitly in each affected section's README.

## Artifacts / key confirmations

| Section | What was verified |
|---|---|
| Docker | Engine 29.6.2, Compose v5.3.1, sudo-less operation, `hello-world` container ran successfully |
| Wazuh stack | All 3 containers healthy; Manager API authenticated; Indexer cluster `status: green` |
| Agent enrollment | `windows-target` agent Active, real SCA/CIS benchmark telemetry flowing |
| FIM | Real-time alerts (rule 550) confirmed on file creation in monitored directories |
| Detection infra | Sysmon Event ID 1 (process creation) and Event ID 3 (network connection) both confirmed logging correctly, full command-line capture verified |
