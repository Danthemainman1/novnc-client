# 30-Minute Quick Start Guide

Get your noVNC remote desktop running in 30 minutes or less.

## Prerequisites

- Ubuntu/Debian server with SSH access
- Sudo privileges
- Port 6443 available

## Step 1: Download Setup Script (1 min)

```bash
# SSH into your server
ssh daniel@192.168.0.53

# Download the setup script
curl -O https://raw.githubusercontent.com/YOUR_USERNAME/novnc-client/main/scripts/server-setup.sh

# Make it executable
chmod +x server-setup.sh
```

## Step 2: Run Setup Script (15 min)

```bash
sudo ./server-setup.sh
```

The script will:
- ✅ Install XFCE4 desktop
- ✅ Install TigerVNC server
- ✅ Generate SSL certificates
- ✅ Install noVNC
- ✅ Configure firewall
- ✅ Set up fail2ban
- ✅ Create systemd services

**When prompted, enter your VNC password.**

## Step 3: Verify Installation (2 min)

```bash
# Check VNC server
sudo systemctl status vncserver@1.service

# Check noVNC
sudo systemctl status novnc.service

# Verify VNC is localhost only
ss -tlnp | grep 5901
# Should show: 127.0.0.1:5901

# Verify noVNC is listening
ss -tlnp | grep 6443
```

## Step 4: Test Local Access (2 min)

From a browser on your local network:

```
https://192.168.0.53:6443
```

1. Accept the certificate warning
2. Enter your VNC password
3. You should see your XFCE desktop!

## Step 5: Configure Router (10 min)

1. Log into your router
2. Create port forward:
   - External Port: 6443
   - Internal IP: 192.168.0.53
   - Internal Port: 6443
   - Protocol: TCP

See [PORT_FORWARDING.md](docs/PORT_FORWARDING.md) for router-specific instructions.

## Step 6: Get Public IP

```bash
curl ifconfig.me
```

## Step 7: Access Remotely

Use the GitHub Pages client: https://YOUR_USERNAME.github.io/novnc-client

Or access directly: https://YOUR_PUBLIC_IP:6443

---

## 🎉 Done!

Your noVNC remote desktop is now accessible from anywhere.

### Quick Reference

| Service | Command |
|---------|---------|
| Start VNC | `sudo systemctl start vncserver@1.service` |
| Stop VNC | `sudo systemctl stop vncserver@1.service` |
| Start noVNC | `sudo systemctl start novnc.service` |
| Stop noVNC | `sudo systemctl stop novnc.service` |
| View logs | `journalctl -u novnc.service -f` |
