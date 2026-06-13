
# New Linux Setup Script
```bash
#!/bin/bash
# ============================================================
# VPS Initial Setup Script
# Tested for: Ubuntu 22.04 / 24.04 (Debian-based)
# Run as root or with sudo: sudo bash vps-setup.sh
# ============================================================

set -e  # exit immediately if any command fails

echo "=============================================="
echo " Starting VPS Setup Script..."
echo "=============================================="

# ------------------------------------------------
# 1. System Update & Basic Tools
# ------------------------------------------------
echo ">>> Updating system packages..."
apt update -y && apt upgrade -y

echo ">>> Installing basic essential tools..."
apt install -y curl wget git unzip software-properties-common \
    build-essential ca-certificates gnupg lsb-release htop \
    ufw fail2ban net-tools

# ------------------------------------------------
# 2. Node.js (LTS) install
# ------------------------------------------------
echo ">>> Installing Node.js LTS..."
curl -fsSL https://deb.nodesource.com/setup_lts.x | bash -
apt install -y nodejs

echo "Node version: $(node -v)"
echo "NPM version: $(npm -v)"

# ------------------------------------------------
# 3. PM2 install (global) + startup config
# ------------------------------------------------
echo ">>> Installing PM2..."
npm install -g pm2

# Enable PM2 to auto-start on system reboot
pm2 startup systemd -u root --hp /root | tail -n 1 > /tmp/pm2_startup_cmd.sh
bash /tmp/pm2_startup_cmd.sh || true

# ------------------------------------------------
# 4. Nginx install
# ------------------------------------------------
echo ">>> Installing Nginx..."
apt install -y nginx
systemctl enable nginx
systemctl start nginx

# ------------------------------------------------
# 5. Certbot (SSL) install — the tool you couldn't remember
# ------------------------------------------------
echo ">>> Installing Certbot (for SSL)..."
apt install -y certbot python3-certbot-nginx

echo "    To issue an SSL certificate later, run:"
echo "    sudo certbot --nginx -d yourdomain.com -d www.yourdomain.com"

# ------------------------------------------------
# 6. Firewall (UFW) basic setup
# ------------------------------------------------
echo ">>> Configuring firewall (UFW)..."
ufw allow OpenSSH
ufw allow 'Nginx Full'   # HTTP (80) + HTTPS (443)
ufw --force enable

# ------------------------------------------------
# 7. Permanent 4GB Swap
# ------------------------------------------------
echo ">>> Setting up 4GB permanent swap..."

if [ -f /swapfile ]; then
    echo "    /swapfile already exists, skipping."
else
    fallocate -l 4G /swapfile
    chmod 600 /swapfile
    mkswap /swapfile
    swapon /swapfile

    # Make swap permanent via fstab entry
    if ! grep -q "/swapfile" /etc/fstab; then
        echo "/swapfile swap swap defaults 0 0" >> /etc/fstab
    fi

    # Tune swappiness (optional, lower value = less aggressive swap usage)
    if ! grep -q "vm.swappiness" /etc/sysctl.conf; then
        echo "vm.swappiness=10" >> /etc/sysctl.conf
        sysctl vm.swappiness=10
    fi
fi

# ------------------------------------------------
# Summary
# ------------------------------------------------
echo "=============================================="
echo " Setup Complete!"
echo "=============================================="
echo "Node.js : $(node -v)"
echo "NPM     : $(npm -v)"
echo "PM2     : $(pm2 -v)"
echo "Nginx   : $(nginx -v 2>&1)"
echo "Certbot : $(certbot --version 2>&1)"
echo "Swap    :"
free -h | grep -i swap
echo "=============================================="
echo "Reboot recommended: sudo reboot"
echo "=============================================="
```