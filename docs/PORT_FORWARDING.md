# Port Forwarding Guide

Configure your router to allow external access to your noVNC server.

## Overview

| Setting | Value |
|---------|-------|
| **External Port** | 6443 |
| **Internal Port** | 6443 |
| **Internal IP** | 192.168.0.53 |
| **Protocol** | TCP |

## Router-Specific Instructions

### TP-Link

1. Log in to **192.168.0.1** or **192.168.1.1**
2. Navigate to: **Advanced** → **NAT Forwarding** → **Virtual Servers**
3. Click **Add**
4. Configure:
   - **Service Type**: noVNC
   - **External Port**: 6443
   - **Internal IP**: 192.168.0.53
   - **Internal Port**: 6443
   - **Protocol**: TCP
5. Click **Save**

---

### Netgear

1. Log in to **routerlogin.net** or **192.168.1.1**
2. Navigate to: **Advanced** → **Advanced Setup** → **Port Forwarding**
3. Click **Add Custom Service**
4. Configure:
   - **Service Name**: noVNC
   - **Protocol**: TCP
   - **External Port Range**: 6443-6443
   - **Internal IP**: 192.168.0.53
   - **Internal Port Range**: 6443-6443
5. Click **Apply**

---

### Linksys

1. Log in to **192.168.1.1**
2. Navigate to: **Security** → **Apps and Gaming** → **Single Port Forwarding**
3. Configure:
   - **Application Name**: noVNC
   - **External Port**: 6443
   - **Internal Port**: 6443
   - **Protocol**: TCP
   - **Device IP**: 192.168.0.53
   - **Enabled**: ✓
4. Click **Save Settings**

---

### ASUS

1. Log in to **router.asus.com** or **192.168.1.1**
2. Navigate to: **WAN** → **Virtual Server / Port Forwarding**
3. Enable **Port Forwarding**
4. Click **Add Profile**
5. Configure:
   - **Service Name**: noVNC
   - **Protocol**: TCP
   - **External Port**: 6443
   - **Internal Port**: 6443
   - **Internal IP**: 192.168.0.53
6. Click **Apply**

---

### D-Link

1. Log in to **192.168.0.1**
2. Navigate to: **Advanced** → **Port Forwarding**
3. Click **Add**
4. Configure:
   - **Name**: noVNC
   - **IP Address**: 192.168.0.53
   - **TCP Port**: 6443-6443
   - **UDP Port**: (leave empty)
5. Click **Save**

---

### Google WiFi / Nest WiFi

1. Open the **Google Home** app
2. Tap **WiFi** → **Settings** → **Advanced networking** → **Port management**
3. Tap **+** to add rule
4. Configure:
   - **Name**: noVNC Desktop
   - **Device**: (Select your server - 192.168.0.53)
   - **External Port**: 6443
   - **Internal Port**: 6443
   - **Protocol**: TCP
5. Tap **Save**

---

### Ubiquiti UniFi

1. Log in to UniFi Controller
2. Navigate to: **Settings** → **Routing & Firewall** → **Port Forwarding**
3. Click **+ Create New Port Forward**
4. Configure:
   - **Name**: noVNC Remote Desktop
   - **From**: Any
   - **Port**: 6443
   - **Forward IP**: 192.168.0.53
   - **Forward Port**: 6443
   - **Protocol**: TCP
5. Click **Save**

---

### pfSense

1. Log in to pfSense webGUI
2. Navigate to: **Firewall** → **NAT** → **Port Forward**
3. Click **+ Add**
4. Configure:
   - **Interface**: WAN
   - **Protocol**: TCP
   - **Destination Port Range**: 6443 to 6443
   - **Redirect Target IP**: 192.168.0.53
   - **Redirect Target Port**: 6443
   - **Description**: noVNC Remote Desktop
5. Click **Save** → **Apply Changes**

---

## Verification

After configuring port forwarding:

1. **Find your public IP**:
   ```
   curl ifconfig.me
   ```

2. **Test external access** from another network:
   ```
   https://YOUR_PUBLIC_IP:6443
   ```

3. **Use online port checker**:
   - Visit [yougetsignal.com/tools/open-ports](https://www.yougetsignal.com/tools/open-ports/)
   - Enter port 6443

## Troubleshooting

### Port still appears closed?

1. **Check internal firewall**: Ensure UFW allows port 6443
   ```bash
   sudo ufw status | grep 6443
   ```

2. **Check noVNC is running**:
   ```bash
   sudo systemctl status novnc.service
   ```

3. **Check IP assignment**: Your server's IP may have changed
   ```bash
   ip addr show
   ```

4. **ISP blocking**: Some ISPs block incoming ports. Contact your ISP or use a VPN/tunnel service.

### Use Static IP

To prevent IP changes, configure a static IP or DHCP reservation:

1. Log in to your router
2. Find DHCP Reservation / Address Reservation
3. Reserve 192.168.0.53 for your server's MAC address
