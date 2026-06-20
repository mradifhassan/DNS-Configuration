---
title: Ultimate Ubuntu Security, Adult Content & Distraction Blocker (2026)
description: Automated script to block NSFW content, major social media distractions, and VPN bypasses on Ubuntu using Cloudflare Family (DoH/DoT) and UFW.
last_updated: June 2026
---

# Ultimate Ubuntu Security & Adult Content Blocker

This project contains an automated shell script to harden Ubuntu system-wide. It filters out adult content, malware, major social media distractions, and implements firewall rules to prevent VPN bypasses.

## Features
* **Automated Deployment**: Single script setup—no tedious manual GUI steps.
* **Encrypted Cloudflare Family DNS**: Uses `1.1.1.3` with **DNS over TLS (DoT)** to block adult sites, phishing, and malware securely.
* **VPN Bypass Prevention**: Blocks standard VPN handshake ports (OpenVPN, WireGuard, IKEv2) via UFW firewall and blocks access to major commercial VPN websites.
* **Distraction Control**: Subtly blocks major social media domains (Facebook, Instagram, TikTok, Twitter/X, Reddit) via the local `/etc/hosts` file.
* **Retains YouTube**: Video learning platforms remain fully accessible.

## Prerequisites
* **OS**: Ubuntu 22.04 LTS or Ubuntu 24.04 LTS (or any Debian-based system using `systemd-resolved` and `NetworkManager`).
* **Privileges**: `root`/`sudo` access required.

---

## How to Use

### Step 0: Copy the script
```bash
#!/bin/bash
# =====================================================
# Ultimate Ubuntu Security & Adult Content Blocker
# Author: Muhammad Radif Hassan
# Last Updated: June 2026
# =====================================================

echo "=== Applying Ultimate Security Hardening ==="

# 1. Enable Firewall and Block Common VPN / Dangerous Ports
echo "[+] Enabling UFW Firewall and blocking VPN ports..."
sudo ufw --force enable

sudo ufw deny out 1194          # OpenVPN
sudo ufw deny out 1194/udp
sudo ufw deny out 51820/udp     # WireGuard
sudo ufw deny out 500           # IKEv2
sudo ufw deny out 4500          # IKEv2 NAT-T
sudo ufw deny out 1701          # L2TP
sudo ufw deny out 1723          # PPTP (old & insecure)
sudo ufw reload

# 2. Strong Porn / Adult Content Blocking via Cloudflare Family (DoH)
echo "[+] Enabling Cloudflare Family DNS (Strong Adult + Malware Blocking)..."
sudo mkdir -p /etc/systemd/resolved.conf.d

sudo tee /etc/systemd/resolved.conf.d/cloudflare-family.conf > /dev/null << DNS
[Resolve]
# Strong Adult Content + Malware + Phishing blocking
DNSOverTLS=yes
DNS=1.1.1.3#family.cloudflare-dns.com 1.0.0.3#family.cloudflare-dns.com
FallbackDNS=1.1.1.3#family.cloudflare-dns.com
Domains=~.
DNSSEC=no
DNS
sudo mkdir -p /etc/NetworkManager/conf.d
echo -e "[main]\ndns=systemd-resolved" | sudo tee /etc/NetworkManager/conf.d/dns.conf

# Fix resolv.conf symlink (very important)
sudo rm -f /etc/resolv.conf
sudo ln -sf /run/systemd/resolve/stub-resolv.conf /etc/resolv.conf

sudo systemctl restart systemd-resolved NetworkManager

# 3. Block Major Distracting & VPN Websites (hosts file)
echo "[+] Blocking distracting and VPN websites..."
sudo tee -a /etc/hosts > /dev/null << BLOCK

# === Security Hardening - Distraction & VPN Blocker ===
# Remove as your preferences
0.0.0.0 facebook.com www.facebook.com 
0.0.0.0 instagram.com www.instagram.com
0.0.0.0 tiktok.com www.tiktok.com
0.0.0.0 snapchat.com www.snapchat.com
0.0.0.0 twitter.com x.com www.twitter.com
0.0.0.0 reddit.com www.reddit.com

# VPN & Proxy Services (Bypass Prevention)
0.0.0.0 nordvpn.com protonvpn.com expressvpn.com surfshark.com mullvad.net
0.0.0.0 privateinternetaccess.com cyberghostvpn.com windscribe.com tunnelbear.com
0.0.0.0 hide.me vpnmentor.com

# Optional: Block more adult-related domains (you can expand this)
0.0.0.0 pornhub.com www.pornhub.com
0.0.0.0 xvideos.com www.xvideos.com
0.0.0.0 xhamster.com www.xhamster.com
0.0.0.0 mat6tube.com www.mat6tube.com
BLOCK

sudo systemctl restart systemd-resolved

echo "============================================"
echo "✅ ULTIMATE SECURITY HARDENING APPLIED!"
echo "• Strong Adult Content Blocking (Cloudflare Family)"
echo "• VPN Bypass Prevention (Ports + Domains)"
echo "• Major Social Media & Distractions Blocked"
echo "• YouTube remains accessible"
echo "============================================"

# Show Status
echo "Current DNS Status:"
resolvectl status | head -n 20
echo ""
echo "Firewall Status:"
sudo ufw status | grep -E "DENY|ALLOW|Active"
```
### Step 1: Create the Script File
Open your terminal and create a new script file:
```bash
# Save as script
sudo nano /usr/local/bin/secure-block.sh
# Paste the whole script above → Ctrl+O → Enter → Ctrl+X

# Make executable
sudo chmod +x /usr/local/bin/secure-block.sh

# Run it
sudo secure-block.sh
