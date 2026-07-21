# SOAR Automation — Cortex + Shuffle

**Status: 🔜 Planned**

Adds Cortex for automated enrichment (IP/domain/hash reputation) and Shuffle for full SOAR workflow automation — tying together every prior module into an end-to-end pipeline: Wazuh alert → Shuffle trigger → `jq`/`tshark` processing → Cortex enrichment → auto-created TheHive case.

Planned sections:
- Cortex deployment and analyzer configuration
- Shuffle deployment and first workflow
- End-to-end pipeline: alert → automated triage → case creation
