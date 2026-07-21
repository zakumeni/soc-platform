# 04 — jq with Real Log Files

Moved from toy JSON to a realistic multi-alert Wazuh export ([`wazuh-sample.json`](./wazuh-sample.json)) — 5 alerts across 3 agents, severity levels ranging 5–12.

## Triage pipeline

```bash
# Extract rule level from every alert
jq '.[] | .rule.level' wazuh-sample.json
# 12 7 12 10 5

# Filter to level 10+ (standard SOC triage cutoff)
jq '.[] | select(.rule.level >= 10)' wazuh-sample.json
# 3 matching alerts

# Reshape into triage-ready objects and save
jq '[.[] | select(.rule.level >= 10) |
     {ip: .data.srcip, rule: .rule.description,
      level: .rule.level, agent: .agent.name}]' \
  wazuh-sample.json > high-severity-alerts.json
```

Result: [`high-severity-alerts.json`](./high-severity-alerts.json) — 3 clean, analyst-ready objects.

## Metrics

```bash
jq 'length' wazuh-sample.json
# 5 — total alerts

jq '[.[] | select(.rule.level >= 10)] | length' wazuh-sample.json
# 3 — high-severity count

jq '[.[] | .data.srcip] | unique | length' wazuh-sample.json
# 3 — unique attacking IPs (185.220.101.5 fired 2 alerts but counts once)
```

## Finding: rule ID collisions break naive grouping

Ran the "noisiest attack type" query:

```bash
jq 'group_by(.rule.id) |
    map({rule: .[0].rule.description, id: .[0].rule.id, count: length}) |
    sort_by(-.count)' wazuh-sample.json
```

Expected the SSH brute force rule to top the frequency ranking. Actual output:

```json
[
  { "rule": "Multiple login failures", "id": 5501, "count": 2 },
  { "rule": "SSH brute force attack",  "id": 5763, "count": 2 },
  { "rule": "Port scan detected",      "id": 100002, "count": 1 }
]
```

**Root cause:** the sample data has two *different* alert descriptions sharing the same `rule.id` (5501) — `"Multiple login failures"` (level 7) and `"Login failure"` (level 5). `group_by(.rule.id)` groups purely by ID, so both collapse into one group of 2. That group's first element (in original array order) is "Multiple login failures," so it reports as the representative description — and since it ties with the SSH rule at count 2, `group_by`'s ascending key sort (5501 before 5763) puts it first.

**Takeaway:** rule IDs aren't guaranteed 1:1 with descriptions in real log data. `group_by(.rule.id)` alone isn't a safe proxy for "attack type" — a production query would need to group by `(.rule.id, .rule.description)` together, or validate that assumption against the actual ruleset before trusting frequency rankings during an incident.

## Full incident summary (single query)

```bash
jq '{
  total_alerts:    length,
  high_severity:   [.[] | select(.rule.level >= 10)] | length,
  unique_attackers:[.[] | .data.srcip] | unique | length,
  attacker_ips:    [.[] | .data.srcip] | unique,
  top_rule:        (group_by(.rule.id) |
                    sort_by(-length) | .[0][0].rule.description)
}' wazuh-sample.json
```

```json
{
  "total_alerts": 5,
  "high_severity": 3,
  "unique_attackers": 3,
  "attacker_ips": ["10.0.0.50", "185.220.101.5", "45.33.32.156"],
  "top_rule": "Multiple login failures"
}
```

(`top_rule` inherits the grouping issue above — flagged here rather than silently "corrected," since surfacing the actual tool output — including its blind spots — is the point of this writeup.)
