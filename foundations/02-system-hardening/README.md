# 02 — System Hardening

Patch the system and install core tooling before adding anything else — this removes known OS vulnerabilities and ensures `apt` can resolve correct dependency versions for every tool installed later.

## Step 1 — Patch the system

```bash
sudo apt update && sudo apt upgrade -y
```

If prompted which services to restart (grub-pc, kernel, etc.), accept the default selections.

## Step 2 — Core CLI essentials

```bash
sudo apt install -y curl wget git vim net-tools build-essential unzip tree
```

These eight packages underpin every tool installed in later modules — downloads, repository additions, and compilation steps depend on them.

## Step 3 — Repository & security prerequisites

```bash
sudo apt install -y apt-transport-https ca-certificates gnupg
```

Required for Ubuntu to validate GPG-signed third-party repos — needed later for Docker's official apt source.

## Step 4 — Verification

```bash
curl --version
wget --version
git --version
vim --version
gcc --version
unzip -v
tree --version
```

All seven confirmed working before proceeding.

## Gotcha encountered

Initial verification returned `curl: command not found` — Steps 2 and 3 hadn't actually been run yet, only checked. Also hit terminal **bracketed-paste garbling** (`^[[200~` prefix, stray `~` suffix on pasted commands) when pasting multi-line blocks — resolved by pasting one command at a time rather than a full multi-line block.
