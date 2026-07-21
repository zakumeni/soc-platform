# 05 — tshark + jq: From Packets to Structured Data

`tshark` is Wireshark's terminal-based capture engine. It reads `.pcap` files (or captures live) and exports every packet as structured JSON, making captures directly parseable by `jq`.

> This section replaces the standard Wireshark GUI capture workflow — the baseline capture below was taken directly via `tshark` on the command line, keeping the whole pipeline scriptable.

## Install

```bash
sudo apt install -y tshark
sudo usermod -aG wireshark $USER
newgrp wireshark
```

## Capture a baseline

```bash
mkdir -p captures
tshark -i ens33 -a duration:30 -w captures/baseline-capture.pcap
```

Result: **690 packets** captured over 30 seconds.

## Export to JSON and inspect structure

```bash
tshark -r captures/baseline-capture.pcap -T json > capture.json
jq '.[0]' capture.json
```

Packet #1 turned out to be the SSH session itself (port 22) — every tshark JSON packet nests under `_source.layers`, with a layer per protocol (`frame`, `eth`, `ip`, `tcp`, and here `ssh`).

## Analysis queries

**Top talkers (source IPs):**
```bash
jq '.[] | ._source.layers.ip["ip.src"]' capture.json | sort | uniq -c | sort -rn
```
```
382 "192.168.171.132"   ← lab VM
380 "192.168.171.1"     ← NAT gateway/host
  4 null                ← ARP broadcasts (no IP layer)
```

**DNS queries (beaconing/C2 hunt):**
```bash
jq -r '.. | .["dns.qry.name"]? // empty' capture.json | sort | uniq -c | sort -rn | head -20
```
No results — the 30-second window only captured an active SSH session, no web/DNS traffic. A quiet baseline is a legitimate result and is reported as such rather than staged to look more eventful.

**Destination ports:**
```bash
jq '.[] | ._source.layers.tcp["tcp.dstport"] // empty' capture.json | sort | uniq -c | sort -rn | head -15
```
```
382 "64722"   ← ephemeral client port
380 "22"      ← SSH server port
```

## Reading

This baseline is entirely explained by one active SSH session — the near-identical 382/380 counts across both the IP and port breakdowns confirm it's the same bidirectional traffic being counted from both directions. A capture with browsing activity in progress would show 443/80 destination ports and populated DNS query names instead.

## Artifacts

- `captures/baseline-capture.pcap` (2.9MB, 690 packets) and `capture.json` (20MB JSON export) were generated locally but are **not committed to this repo** — raw packet captures can contain real local network metadata (MAC addresses, internal IPs) and the JSON export is large. The queries and results above are fully reproducible by re-running the commands in this README against your own capture.

## Checkpoint

VM snapshot taken (`Day1-jq-Wireshark-Complete`) before proceeding to the SIEM Deployment module, which installs Docker and modifies system-level services.
