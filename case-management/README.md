# Case Management — Wazuh Tuning + DFIR-IRIS

**Status: 🔜 Planned**

Tunes Wazuh detection rules and deploys **DFIR-IRIS** for structured incident case management.

> Originally planned around TheHive, switched to DFIR-IRIS — an actively-developed, fully open-source collaborative incident response platform. Docker-native deployment, API-first design, and (unlike TheHive's more recent licensing shift toward a closed-source model for newer versions) stays open source.

Planned sections:
- Custom Wazuh rule authoring
- DFIR-IRIS deployment via Docker Compose
- Case template setup and manual case creation from a Wazuh alert
- Cortex integration for IOC enrichment (VirusTotal, AbuseIPDB, etc.) within IRIS cases
