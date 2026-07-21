# 01 — Lab Setup

Built the analyst workstation as a VM in VMware Workstation.

## VM Configuration

| Setting | Value | Why |
|---|---|---|
| Name | `BlueTeam-Workstation` | Matches lab topology naming used throughout the platform |
| Guest OS | Ubuntu 22.04.5 LTS (64-bit) | LTS stability, wide tool compatibility |
| Hardware compatibility | Workstation 25H2 or later | Also flagged ESX Server-compatible, in case the VM is later migrated to a bare-metal ESXi host |
| CPU | 4 vCPU | Headroom for later Docker-based services (Wazuh, TheHive, Cortex, Shuffle all run concurrently by the final module) |
| Memory | 8192 MB | Wazuh alone typically needs 4GB+; multiple JVM-based services later in the build need real headroom above Ubuntu's own 6GB recommendation |
| Disk | 100GB, thin-provisioned, split into multiple files | Docker images for the full stack can easily consume 15-20GB+ on top of the base OS and capture files |
| Network | NAT | VM needs outbound internet for `apt` installs and Docker image pulls; NAT avoids router/firewall configuration required by Bridged mode |
| SCSI Controller | LSI Logic (default) | Standard, broadly compatible; Paravirtualized SCSI requires VMware Tools pre-installed, which creates a chicken-and-egg problem during initial install |

## Install method

Used VMware's **Easy Install** with Ubuntu's Subiquity installer. Note: Subiquity's installer is a keyboard-only, ncurses-style interface at some stages — no mouse cursor renders during install, which is expected behavior, not a bug. Navigation is via Tab / Arrow keys / Enter.

## SSH access

Installed and enabled OpenSSH server post-install to allow working from the host terminal instead of the VMware console window:

```bash
sudo apt update
sudo apt install openssh-server -y
sudo systemctl enable --now ssh
ip a   # find the VM's NAT IP to connect to
```

```bash
ssh <username>@<vm-ip>
```
