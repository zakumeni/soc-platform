# 03 — jq Fundamentals

`jq` is a lightweight, zero-dependency command-line JSON processor. In a modern SOC, almost every tool — Wazuh, TheHive, Shuffle, Cortex, and most SIEM APIs — outputs JSON. `jq` parses, filters, reshapes, and extracts that data directly in the terminal.

## Install

```bash
sudo apt install -y jq
jq --version   # jq-1.6
```

## Core patterns practiced

| Filter | What it does |
|---|---|
| `jq .` | Pretty-print raw JSON (identity filter) |
| `jq '.field'` | Extract a single top-level field |
| `jq '.nested.field'` | Navigate nested objects (e.g. `.rule.description`) |
| `jq '{a: .x, b: .y}'` | Object construction — reshape JSON into only the fields needed |
| `jq '.[]'` | Iterate — explode an array into individual elements |
| `jq '.[] \| select(cond)'` | Chain iteration with filtering — keep only matching elements |
| `>>` / `>` | Append/overwrite output to a file — building IOC lists and extracted artifacts |

## Example: nested field extraction

`sample-alert.json`:
```json
{
  "timestamp": "2024-01-15T08:23:41Z",
  "src_ip": "192.168.1.100",
  "rule": {
    "id": 5763,
    "description": "SSH brute force attempt",
    "level": 10
  }
}
```

```bash
jq '.rule.description' sample-alert.json
# "SSH brute force attempt"

jq '{ip: .src_ip, level: .rule.level, desc: .rule.description}' sample-alert.json
# {"ip": "192.168.1.100", "level": 10, "desc": "SSH brute force attempt"}
```

## Example: array + select() triage

`alerts.json` — 3 alerts, 2 high severity, 1 medium:

```bash
jq '.[] | select(.severity == "high") | .src_ip' alerts.json >> extracted-iocs.txt
cat extracted-iocs.txt
# "10.0.0.1"
# "10.0.0.3"
```

This `select()` + append pattern is the foundation of automated triage — surface only what matters, discard the noise, and build a running IOC list.

See [`sample-alert.json`](./sample-alert.json), [`alerts.json`](./alerts.json), and [`extracted-iocs.txt`](./extracted-iocs.txt) for the exact files used.
