# Setup Checklist

Track your progress through the noVNC setup.

---

## Server Setup

### System Preparation
- [ ] SSH connection established
- [ ] System packages updated

### Desktop Environment
- [ ] XFCE4 installed
- [ ] XFCE4 goodies installed

### VNC Server
- [ ] TigerVNC installed
- [ ] ~/.vnc directory created
- [ ] xstartup script created
- [ ] VNC config file created
- [ ] VNC password set

### SSL/TLS Certificates
- [ ] /etc/novnc directory created
- [ ] 3072-bit RSA key generated
- [ ] Self-signed certificate generated
- [ ] Permissions set (644/600)

### noVNC
- [ ] noVNC cloned to /opt/novnc
- [ ] websockify cloned to /opt/novnc/utils/websockify

### Systemd Services
- [ ] vncserver@.service created
- [ ] novnc.service created
- [ ] Services enabled for boot
- [ ] VNC service started
- [ ] noVNC service started

### Firewall (UFW)
- [ ] UFW installed
- [ ] Default deny incoming
- [ ] Default allow outgoing
- [ ] SSH (22/tcp) allowed
- [ ] noVNC (6443/tcp) allowed
- [ ] VNC (5901/tcp) denied
- [ ] UFW enabled

### Brute-Force Protection
- [ ] fail2ban installed
- [ ] noVNC filter created
- [ ] SSH jail configured (3 attempts, 1hr)
- [ ] noVNC jail configured (5 attempts, 1hr)
- [ ] fail2ban enabled

---

## Verification

### Network Binding
- [ ] VNC listening on 127.0.0.1:5901 only
- [ ] noVNC listening on port 6443

### Service Status
- [ ] vncserver@1.service active
- [ ] novnc.service active
- [ ] fail2ban.service active

### Firewall Status
- [ ] UFW active and showing correct rules

### Connection Test
- [ ] Local HTTPS test successful
- [ ] Browser connection working
- [ ] VNC password authentication working
- [ ] Desktop visible

---

## GitHub Pages

- [ ] Repository created
- [ ] index.html uploaded
- [ ] app/ directory with noVNC client
- [ ] README.md uploaded
- [ ] docs/SECURITY.md uploaded
- [ ] docs/PORT_FORWARDING.md uploaded
- [ ] QUICKSTART.md uploaded
- [ ] EXECUTION_GUIDE.md uploaded
- [ ] GitHub Pages enabled
- [ ] Site accessible

---

## Router Configuration

- [ ] Port forwarding rule created
- [ ] External port: 6443
- [ ] Internal IP: 192.168.0.53
- [ ] Internal port: 6443
- [ ] Protocol: TCP
- [ ] Rule enabled

---

## Final Testing

- [ ] Public IP obtained
- [ ] External access verified
- [ ] GitHub Pages URL working
- [ ] End-to-end connection successful

---

## Completion

- [ ] All services running
- [ ] All security measures active
- [ ] Documentation complete
- [ ] External access confirmed
