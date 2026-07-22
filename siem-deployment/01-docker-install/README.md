# 01 — Docker Installation

Installed Docker Engine and Compose from Docker's official apt repository (not Ubuntu's default `docker.io` package, which lags behind).

## Steps

```bash
# Remove any conflicting old packages (safe even if none installed)
sudo apt remove -y docker docker-engine docker.io containerd runc

# Add Docker's official GPG key + repo
sudo apt update && sudo apt install -y ca-certificates curl gnupg
sudo install -m 0755 -d /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] \
  https://download.docker.com/linux/ubuntu $(. /etc/os-release && echo $VERSION_CODENAME) stable" \
  | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

# Install Docker Engine + Compose plugin
sudo apt update
sudo apt install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin

# Run without sudo
sudo usermod -aG docker $USER
newgrp docker

# Enable on boot
sudo systemctl enable docker
sudo systemctl enable containerd
```

## Verification

```bash
docker --version          # Docker version 29.6.2, build dfc4efb
docker compose version    # Docker Compose version v5.3.1
docker run hello-world    # "Hello from Docker!"
docker pull alpine:latest # confirms Hub connectivity, runs sudo-less
sudo systemctl status docker  # Active: active (running)
```

## Gotcha encountered

`sudo apt remove -y docker docker-engine docker.io containerd runc` failed outright with `E: Unable to locate package docker-engine` — a genuinely nonexistent package name in this repo causes the **entire remove command to abort**, unlike "package not installed" which apt skips gracefully. Harmless here (fresh VM, nothing to remove), but worth knowing: a failed multi-package apt command doesn't mean none of it ran — it can mean none of it ran.
