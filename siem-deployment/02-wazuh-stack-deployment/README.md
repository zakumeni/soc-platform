# 02 — Wazuh Stack Deployment via Docker Compose

Deployed the full Wazuh 4.9.2 stack (Manager, Indexer, Dashboard) as three containers using the official `wazuh-docker` repository, pinned to the `v4.9.2` tag.

## Steps

```bash
# OpenSearch (Indexer) requires this raised, or the container OOMs on startup
sudo sysctl -w vm.max_map_count=262144
echo 'vm.max_map_count=262144' | sudo tee -a /etc/sysctl.conf
sudo sysctl -p

# Clone pinned to a specific tag, not main
git clone https://github.com/wazuh/wazuh-docker.git -b v4.9.2
cd wazuh-docker/single-node

# Generate SSL certs for mutual TLS between the three services
docker compose -f generate-indexer-certs.yml run --rm generator
sudo chown -R 1000:1000 config/wazuh_indexer_ssl_certs/
sudo chmod -R 755 config/wazuh_indexer_ssl_certs/

# Launch the stack
docker compose up -d
```

## Verification

**All 3 containers healthy:**
```bash
docker ps
# single-node-wazuh.manager-1, single-node-wazuh.indexer-1, single-node-wazuh.dashboard-1 — all Up
```

**Dashboard login:** `https://<vm-ip>` — `admin` / `SecretPassword` (default from `docker-compose.yml`)

**Manager API health check:**
```bash
TOKEN=$(docker exec single-node-wazuh.manager-1 \
  curl -k -u wazuh-wui:'MyS3cr37P450r.*-' \
  https://localhost:55000/security/user/authenticate?raw=true)

docker exec -it single-node-wazuh.manager-1 \
  curl -k -H "Authorization: Bearer $TOKEN" https://localhost:55000/
# {"data": {"title": "Wazuh API REST", "api_version": "4.9.2", ...}, "error": 0}
```

**Indexer cluster health:**
```bash
docker exec -it single-node-wazuh.indexer-1 curl -k -u admin:SecretPassword https://localhost:9200/_cluster/health
# "status":"green", 11/11 active shards
```

## Analyst account

Created a dedicated `apiuser` (administrator role) via Dashboard → Security → Users, separate from the built-in `wazuh` / `wazuh-wui` service accounts.
