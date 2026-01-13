# Security Documentation

## Architecture Overview

```
┌─────────────────────────────────────────────────────────────────┐
│                        INTERNET                                  │
└─────────────────────────────────────────────────────────────────┘
                              │
                              │ Port 6443 (HTTPS/WSS)
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│                   ROUTER/FIREWALL                                │
│              (Port Forward 6443 → Server)                        │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│                    UFW FIREWALL                                  │
│   ┌────────────┐  ┌────────────┐  ┌────────────────────────┐    │
│   │  ALLOW 22  │  │ ALLOW 6443 │  │     DENY 5901          │    │
│   │   (SSH)    │  │  (noVNC)   │  │ (Block Direct VNC)     │    │
│   └────────────┘  └────────────┘  └────────────────────────┘    │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│                 UBUNTU SERVER (192.168.0.53)                     │
│                                                                  │
│  ┌──────────────────────────────────────────────────────────┐   │
│  │                  noVNC Proxy (:6443)                      │   │
│  │              SSL/TLS Termination                          │   │
│  │              (cert.pem + key.pem)                         │   │
│  └──────────────────────────────────────────────────────────┘   │
│                              │                                   │
│                              │ WebSocket → VNC (localhost only)  │
│                              ▼                                   │
│  ┌──────────────────────────────────────────────────────────┐   │
│  │              TigerVNC Server (:5901)                      │   │
│  │              Bound to 127.0.0.1 ONLY                      │   │
│  │              (NOT accessible from network)                │   │
│  └──────────────────────────────────────────────────────────┘   │
│                              │                                   │
│                              ▼                                   │
│  ┌──────────────────────────────────────────────────────────┐   │
│  │                  XFCE4 Desktop                            │   │
│  └──────────────────────────────────────────────────────────┘   │
│                                                                  │
│  ┌──────────────────────────────────────────────────────────┐   │
│  │                    fail2ban                               │   │
│  │     SSH: 3 attempts → 1hr ban                             │   │
│  │   noVNC: 5 attempts → 1hr ban                             │   │
│  └──────────────────────────────────────────────────────────┘   │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

## Security Measures

### 1. Encryption (SSL/TLS)

All traffic between browser and server is encrypted using:
- **3072-bit RSA** key
- **Self-signed certificate** (365-day validity)
- **SSL-only mode** (unencrypted connections rejected)

```bash
# Certificate location
/etc/novnc/cert.pem  (public certificate)
/etc/novnc/key.pem   (private key, restricted)
```

### 2. Network Isolation

VNC server is bound to **localhost only**:

```bash
# VNC listens ONLY on 127.0.0.1:5901
vncserver :1 -localhost yes
```

This means:
- ✅ noVNC proxy can connect (same machine)
- ❌ External connections to port 5901 are impossible

### 3. Firewall Rules

```bash
# Active UFW rules
sudo ufw status verbose

To                         Action      From
--                         ------      ----
22/tcp                     ALLOW       Anywhere    # SSH
6443/tcp                   ALLOW       Anywhere    # noVNC HTTPS
5901/tcp                   DENY        Anywhere    # Block VNC
```

### 4. Brute-Force Protection

fail2ban monitors for authentication failures:

| Service | Max Attempts | Ban Duration |
|---------|--------------|--------------|
| SSH     | 3            | 1 hour       |
| noVNC   | 5            | 1 hour       |

```bash
# Check ban status
sudo fail2ban-client status sshd
sudo fail2ban-client status novnc

# Unban an IP
sudo fail2ban-client set sshd unbanip 192.168.1.100
```

## Monitoring Commands

```bash
# Service status
sudo systemctl status vncserver@1.service
sudo systemctl status novnc.service

# Active connections
ss -tlnp | grep -E '5901|6443'

# Firewall logs
sudo tail -f /var/log/ufw.log

# fail2ban logs
sudo tail -f /var/log/fail2ban.log

# View banned IPs
sudo fail2ban-client status sshd
```

## Best Practices

1. **Use strong VNC password** (12+ characters, mixed case, numbers, symbols)
2. **Regularly update** system packages: `sudo apt update && sudo apt upgrade`
3. **Monitor logs** for suspicious activity
4. **Consider IP whitelisting** for additional security
5. **Use port forwarding** only for port 6443, never 5901

## Threat Model

### Protected Against

- ✅ Man-in-the-middle attacks (SSL/TLS encryption)
- ✅ Direct VNC access (localhost binding + firewall)
- ✅ Brute-force attacks (fail2ban)
- ✅ Unauthorized network access (UFW firewall)

### Known Limitations

- ⚠️ **Self-signed certificate**: Browser shows warning (but connection is still encrypted)
- ⚠️ **Password authentication only**: No 2FA support in noVNC
- ⚠️ **Single-user VNC**: All connections share the same desktop

## Emergency Procedures

### Lock Down Server

```bash
# Block all external access except SSH
sudo ufw default deny incoming
sudo ufw allow 22/tcp
sudo ufw reload
```

### Stop All VNC Services

```bash
sudo systemctl stop novnc.service
sudo systemctl stop vncserver@1.service
```

### View Recent Attacks

```bash
# Check fail2ban banned IPs
sudo fail2ban-client status sshd
sudo fail2ban-client status novnc

# Check auth log for attack patterns
sudo grep "Failed" /var/log/auth.log | tail -50
```
