# Execution Guide

Detailed step-by-step instructions with verification at each stage.

---

## Phase 1: Server Preparation

### 1.1 SSH Connection

```bash
ssh daniel@192.168.0.53
```

**Verification**: You should see a command prompt like `daniel@hostname:~$`

### 1.2 System Update

```bash
sudo apt update && sudo apt upgrade -y
```

**Verification**: 
```bash
apt list --upgradable
# Should show: "Listing... Done" with no packages
```

---

## Phase 2: Desktop Environment

### 2.1 Install XFCE4

```bash
sudo apt install -y xfce4 xfce4-goodies
```

**Verification**:
```bash
dpkg -l | grep xfce4
# Should list xfce4 packages
```

---

## Phase 3: VNC Server

### 3.1 Install TigerVNC

```bash
sudo apt install -y tigervnc-standalone-server tigervnc-common
```

**Verification**:
```bash
vncserver --version
# Should show TigerVNC version
```

### 3.2 Create VNC Configuration

```bash
mkdir -p ~/.vnc

# Create xstartup
cat > ~/.vnc/xstartup << 'EOF'
#!/bin/bash
unset SESSION_MANAGER
unset DBUS_SESSION_BUS_ADDRESS
export XKL_XMODMAP_DISABLE=1
exec startxfce4
EOF

chmod +x ~/.vnc/xstartup

# Create config file
cat > ~/.vnc/config << 'EOF'
geometry=1920x1080
depth=24
localhost=yes
EOF
```

### 3.3 Set VNC Password

```bash
vncpasswd
# Enter and confirm your password
```

**Verification**:
```bash
ls -la ~/.vnc/
# Should show: xstartup, config, passwd
```

---

## Phase 4: SSL Certificates

### 4.1 Generate Certificates

```bash
sudo mkdir -p /etc/novnc

sudo openssl req -x509 -nodes -newkey rsa:3072 \
    -keyout /etc/novnc/key.pem \
    -out /etc/novnc/cert.pem \
    -days 365 \
    -subj "/C=US/ST=State/L=City/O=noVNC/OU=Remote Desktop/CN=192.168.0.53"

sudo chmod 644 /etc/novnc/cert.pem
sudo chmod 600 /etc/novnc/key.pem
```

**Verification**:
```bash
openssl x509 -in /etc/novnc/cert.pem -text -noout | head -20
# Should show certificate details
```

---

## Phase 5: noVNC Installation

### 5.1 Clone Repositories

```bash
sudo git clone https://github.com/novnc/noVNC.git /opt/novnc
sudo git clone https://github.com/novnc/websockify.git /opt/novnc/utils/websockify
```

**Verification**:
```bash
ls /opt/novnc/
# Should show noVNC files including vnc.html

ls /opt/novnc/utils/websockify/
# Should show websockify files
```

---

## Phase 6: Systemd Services

### 6.1 Create VNC Service

```bash
sudo tee /etc/systemd/system/vncserver@.service << 'EOF'
[Unit]
Description=TigerVNC Server for display %i
After=syslog.target network.target

[Service]
Type=simple
User=daniel
Group=daniel
WorkingDirectory=/home/daniel
ExecStart=/usr/bin/vncserver :1 -localhost yes -geometry 1920x1080 -depth 24 -fg
ExecStop=/usr/bin/vncserver -kill :1
Restart=on-failure
RestartSec=10

[Install]
WantedBy=multi-user.target
EOF
```

### 6.2 Create noVNC Service

```bash
sudo tee /etc/systemd/system/novnc.service << 'EOF'
[Unit]
Description=noVNC SSL Proxy Service
After=syslog.target network.target vncserver@1.service
Requires=vncserver@1.service

[Service]
Type=simple
User=root
WorkingDirectory=/opt/novnc
ExecStart=/opt/novnc/utils/novnc_proxy --listen 6443 --vnc localhost:5901 --cert /etc/novnc/cert.pem --key /etc/novnc/key.pem --ssl-only
Restart=on-failure
RestartSec=10

[Install]
WantedBy=multi-user.target
EOF
```

### 6.3 Enable Services

```bash
sudo systemctl daemon-reload
sudo systemctl enable vncserver@1.service
sudo systemctl enable novnc.service
```

**Verification**:
```bash
systemctl is-enabled vncserver@1.service
systemctl is-enabled novnc.service
# Both should show: enabled
```

---

## Phase 7: Firewall

### 7.1 Configure UFW

```bash
sudo apt install -y ufw

sudo ufw default deny incoming
sudo ufw default allow outgoing
sudo ufw allow 22/tcp comment 'SSH'
sudo ufw allow 6443/tcp comment 'noVNC HTTPS'
sudo ufw deny 5901/tcp comment 'Block direct VNC'
sudo ufw --force enable
```

**Verification**:
```bash
sudo ufw status verbose
# Should show rules for 22, 6443 (ALLOW), 5901 (DENY)
```

---

## Phase 8: fail2ban

### 8.1 Install and Configure

```bash
sudo apt install -y fail2ban

# Create noVNC filter
sudo tee /etc/fail2ban/filter.d/novnc.conf << 'EOF'
[Definition]
failregex = ^.*connection from <HOST>.*failed.*$
            ^.*<HOST>.*authentication failure.*$
ignoreregex =
EOF

# Create jail configuration
sudo tee /etc/fail2ban/jail.local << 'EOF'
[DEFAULT]
bantime = 3600
findtime = 600
maxretry = 5
backend = systemd

[sshd]
enabled = true
port = ssh
filter = sshd
logpath = /var/log/auth.log
maxretry = 3
bantime = 3600

[novnc]
enabled = true
port = 6443
filter = novnc
logpath = /var/log/syslog
maxretry = 5
bantime = 3600
EOF

sudo systemctl restart fail2ban
sudo systemctl enable fail2ban
```

**Verification**:
```bash
sudo fail2ban-client status
# Should show sshd and novnc jails
```

---

## Phase 9: Start Services

### 9.1 Start Everything

```bash
# Kill any existing VNC session
vncserver -kill :1 2>/dev/null || true

# Start services
sudo systemctl start vncserver@1.service
sleep 3
sudo systemctl start novnc.service
```

**Verification**:
```bash
sudo systemctl status vncserver@1.service
sudo systemctl status novnc.service
# Both should show: active (running)
```

---

## Phase 10: Final Verification

### 10.1 Network Check

```bash
# VNC should be localhost only
ss -tlnp | grep 5901
# Expected: 127.0.0.1:5901

# noVNC should be listening on all interfaces
ss -tlnp | grep 6443
# Expected: *:6443 or 0.0.0.0:6443
```

### 10.2 Test Connection

```bash
curl -k https://localhost:6443
# Should return HTML content
```

### 10.3 Browser Test

Open in browser: `https://192.168.0.53:6443`

1. Accept certificate warning
2. Enter VNC password
3. Desktop should appear

---

## 🎉 Setup Complete!

Your noVNC server is ready. Configure port forwarding on your router to enable remote access.
