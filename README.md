# noVNC Remote Desktop Client

Secure, web-based remote desktop access using noVNC with SSL/TLS encryption.

## 🚀 Quick Start

1. **Enter your server address** (IP or hostname)
2. **Click "Connect to Desktop"**
3. **Accept the certificate warning** (first time only)
4. **Enter your VNC password**

## 🔒 Security Features

- **SSL/TLS Encryption**: All traffic encrypted with 3072-bit RSA
- **Localhost VNC Binding**: VNC server only accessible via localhost
- **Firewall Protected**: Only port 6443 exposed externally
- **Brute-Force Protection**: fail2ban protects against attacks

## 📖 Documentation

- [Security Documentation](docs/SECURITY.md) - Architecture and security measures
- [Port Forwarding Guide](docs/PORT_FORWARDING.md) - Router configuration

## 🔧 Server Requirements

- Ubuntu/Debian Linux
- XFCE4 Desktop Environment
- TigerVNC Server
- noVNC with websockify
- UFW Firewall
- fail2ban

## 📋 Quick Commands

```bash
# Check VNC status
sudo systemctl status vncserver@1.service

# Check noVNC status
sudo systemctl status novnc.service

# Check firewall
sudo ufw status

# Check fail2ban
sudo fail2ban-client status
```

## ⚠️ Certificate Warning

This setup uses self-signed certificates. Your browser will show a security warning on first connection. This is expected - click "Advanced" → "Proceed" to continue.

## 📄 License

MIT License - See [noVNC License](https://github.com/novnc/noVNC/blob/master/LICENSE.txt)
