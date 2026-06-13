# Swap Setup Guide
```bash
#!/bin/bash
# ============================================================
# 4GB Permanent Swap Setup Script
# Run as root or with sudo: sudo bash swap-setup.sh
# ============================================================

set -e

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

echo "=============================================="
echo " Swap Setup Complete!"
echo "=============================================="
free -h | grep -i swap
```