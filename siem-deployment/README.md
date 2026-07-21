# SIEM Deployment — Docker + Wazuh

**Status: 🔜 Planned**

Deploys Docker Engine and Docker Compose, then brings up the Wazuh stack (Manager, Indexer, Dashboard) as containers. The lab VM is enrolled as a monitored Wazuh agent, and the first live alerts start flowing.

Planned sections:
- Docker Engine + Compose installation
- Wazuh all-in-one stack deployment via `docker-compose`
- Wazuh agent install on `BlueTeam-Workstation`
- Dashboard walkthrough and first live alert review
- Re-running the [Foundations `jq` queries](../foundations/04-jq-real-log-triage/) against real Wazuh output instead of the simulated `wazuh-sample.json`
