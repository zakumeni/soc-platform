# 06 — Sniffnet

`Sniffnet` is a free, open-source, Rust-based traffic monitor — a lighter-weight, visual complement to the `tshark`/`jq` CLI pipeline. Good for quick sanity checks: active connections, bandwidth spikes, and unknown remote hosts at a glance, without writing a query.

## Install (Ubuntu)

```bash
sudo apt update
sudo apt install libpcap-dev libasound2-dev libfontconfig1-dev libgtk-3-dev libwayland-dev libxkbcommon-dev

cd ~/Downloads
wget https://github.com/GyulyVGC/sniffnet/releases/latest/download/Sniffnet_LinuxDEB_amd64.deb
sudo apt install ./Sniffnet_LinuxDEB_amd64.deb
```

## Launch

```bash
sudo sniffnet
```
`sudo` is required (unlike Wireshark's group-based capture) since Sniffnet needs elevated privileges to enumerate network adapters directly.

## Role in the platform

Sniffnet isn't part of the automated pipeline (Wazuh/TheHive/Cortex/Shuffle don't integrate with it) — it's kept in the toolkit as a fast manual verification tool: when an automated alert fires, a quick look at Sniffnet's live dashboard can confirm whether the suspicious traffic is still active before diving into a full `tshark`/`jq` investigation.
