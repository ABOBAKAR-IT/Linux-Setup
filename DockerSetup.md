# Docker Setup on Linux
```bash
#!/bin/bash
# ============================================================
# Docker & Docker Compose Installation Script (Latest Version)
# Tested for: Ubuntu 22.04 / 24.04 (Debian-based)
# Run as root or with sudo: sudo bash docker-setup.sh
# ============================================================

set -e

echo "=============================================="
echo " Starting Docker Installation..."
echo "=============================================="

# ------------------------------------------------
# 1. Remove old/conflicting Docker packages (if any)
# ------------------------------------------------
echo ">>> Removing old Docker packages (if present)..."
for pkg in docker.io docker-doc docker-compose docker-compose-v2 podman-docker containerd runc; do
    apt-get remove -y $pkg >/dev/null 2>&1 || true
done

# ------------------------------------------------
# 2. Install prerequisites
# ------------------------------------------------
echo ">>> Installing prerequisites..."
apt-get update -y
apt-get install -y ca-certificates curl gnupg

# ------------------------------------------------
# 3. Add Docker's official GPG key
# ------------------------------------------------
echo ">>> Adding Docker's official GPG key..."
install -m 0755 -d /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
chmod a+r /etc/apt/keyrings/docker.asc

# ------------------------------------------------
# 4. Add Docker repository
# ------------------------------------------------
echo ">>> Adding Docker repository..."
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
  tee /etc/apt/sources.list.d/docker.list > /dev/null

# ------------------------------------------------
# 5. Install Docker Engine + Compose plugin
# ------------------------------------------------
echo ">>> Installing Docker Engine, CLI, Containerd, and Compose plugin..."
apt-get update -y
apt-get install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin

# ------------------------------------------------
# 6. Enable Docker service
# ------------------------------------------------
echo ">>> Enabling and starting Docker service..."
systemctl enable docker
systemctl start docker

# ------------------------------------------------
# 7. Allow current non-root user to run Docker (optional)
# ------------------------------------------------
if [ -n "$SUDO_USER" ]; then
    echo ">>> Adding user '$SUDO_USER' to the docker group..."
    usermod -aG docker "$SUDO_USER"
    echo "    Log out and log back in for this to take effect."
fi

# ------------------------------------------------
# Summary
# ------------------------------------------------
echo "=============================================="
echo " Docker Installation Complete!"
echo "=============================================="
docker --version
docker compose version
echo "=============================================="
```
