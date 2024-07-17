# FriendlyElec CM3588 SBC with NAS Kit - Setup Guide

![FriendlyElec CM3588 SBC](https://github.com/user-attachments/assets/3b0c5de8-405d-4200-b5da-cfa08722fcac)

## Table of Contents

1. [Introduction](#introduction)
2. [Initial Setup](#initial-setup)
3. [System Configuration](#system-configuration)
   - [Package Management](#package-management)
   - [Kernel and Firmware](#kernel-and-firmware)
   - [CPU Fan Configuration](#cpu-fan-configuration)
4. [Network Configuration](#network-configuration)
   - [Wi-Fi Setup](#wi-fi-setup)
5. [System Optimization](#system-optimization)
   - [Localization](#localization)
   - [Security Enhancements](#security-enhancements)
6. [Remote Access](#remote-access)
   - [SSH Configuration](#ssh-configuration)
   - [VNC Server Setup](#vnc-server-setup)
   - [Tailscale VPN Installation](#tailscale-vpn-installation)
7. [NAS Functionality](#nas-functionality)
   - [CasaOS Installation](#casaos-installation)
   - [HTTPS Configuration with Tailscale and Caddy](#https-configuration-with-tailscale-and-caddy)
8. [Advanced Topics](#advanced-topics)
   - [Custom DNS Configuration](#custom-dns-configuration)
9. [Troubleshooting](#troubleshooting)
10. [Additional Resources](#additional-resources)

## Introduction

The FriendlyElec CM3588 Single Board Computer (SBC) with NAS Kit is a versatile platform for creating a powerful Network Attached Storage (NAS) solution. This comprehensive guide covers the setup process, including installation of Debian Desktop (Bullseye), essential service configurations, and system optimizations for NAS functionality.


## Initial Setup

1. Download the latest Debian Desktop (Bullseye) image for CM3588 from the [FriendlyElec website](https://www.friendlyelec.com/index.php?route=product/product&product_id=287).
2. Use [BalenaEtcher](https://www.balena.io/etcher/) to create a bootable SD card with the image.
3. Insert the SD card into the CM3588 and power on the device.
4. Allow the system to complete the initial boot process and flash the eMMC.
5. Remove the SD card and reboot the device.

## System Configuration

### Package Management

Update the package sources to ensure access to the latest software:

1. SSH into the device using the default credentials (username: `pi`, password: `pi`).
2. Update the package sources:

```bash
sudo mv /etc/apt/sources.list /etc/apt/sources.list.old
sudo nano /etc/apt/sources.list
```

3. Add the following content to the new `sources.list` file:

```
deb http://deb.debian.org/debian bookworm main non-free-firmware
deb-src http://deb.debian.org/debian bookworm main non-free-firmware
deb http://deb.debian.org/debian-security/ bookworm-security main non-free-firmware
deb-src http://deb.debian.org/debian-security/ bookworm-security main non-free-firmware
deb http://deb.debian.org/debian bookworm-updates main non-free-firmware
deb-src http://deb.debian.org/debian bookworm-updates main non-free-firmware
deb http://deb.debian.org/debian bookworm-backports main non-free-firmware
deb-src http://deb.debian.org/debian bookworm-backports main non-free-firmware
```

4. Update the package lists:

```bash
sudo apt-get update
```

### Kernel and Firmware

Install necessary kernel headers and proprietary firmware:

1. Install kernel headers:

```bash
sudo -i
dpkg -i /opt/archives/linux-headers-6.1.57_6.1.57-13_arm64.deb
```

2. Install and update USB Wi-Fi driver firmware:

```bash
sudo apt update
sudo apt install firmware-realtek
sudo apt upgrade firmware-realtek
sudo modprobe -r r8188eu
sudo modprobe r8188eu
dmesg | grep r8188eu
sudo reboot
```

### CPU Fan Configuration

To address potential CPU fan issues, follow these steps:

1. Open the GPIO configuration file:

```bash
sudo nano /boot/rk3588.dtb
```

2. Locate the `fan` section and modify it as follows:

```
fan: pwm-fan {
    compatible = "pwm-fan";
    #cooling-cells = <2>;
    pwms = <&pwm14 0 50000 0>;
    cooling-levels = <0 36 36 36 36 36 36 36 36 36 36 36 36 36 36 36 36 36 36 36 36 36 36 36 36 36 36 36 36 36 36 36 36 36 36 36 36 36 36 36 36 36 36 36 36 36 36 36 36 36 36 36 36 36 36 36 36 36 36 36 36 36 36 36 255>;
    status = "okay";
};
```

3. Save the file and reboot the system:

```bash
sudo reboot
```

For more details on this fix, refer to the [FriendlyElec forum thread](https://www.friendlyelec.com/Forum/viewtopic.php?p=14618#p14300).

## Network Configuration

### Wi-Fi Setup

Configure Wi-Fi for network connectivity:

1. List available Wi-Fi access points:

```bash
nmcli device wifi list
```

2. Connect to a Wi-Fi network:

```bash
sudo nmcli device wifi connect "SSID" password "PASSWORD"
```

3. Set auto-connect for the Wi-Fi network:

```bash
sudo nmcli connection modify "SSID" connection.autoconnect yes
```

4. Set connection priority (optional):

```bash
sudo nmcli connection modify "SSID" connection.autoconnect-priority 1
```

## System Optimization

### Localization

Set the correct timezone and locale for your region:

1. Set timezone:

```bash
sudo timedatectl set-timezone Your/Timezone
```

2. Configure locale:

```bash
sudo dpkg-reconfigure locales
```

3. Add locale environment variables to `.bashrc`:

```bash
echo "export LC_ALL=en_US.UTF-8" >> ~/.bashrc
echo "export LANG=en_US.UTF-8" >> ~/.bashrc
echo "export LANGUAGE=en_US.UTF-8" >> ~/.bashrc
```

4. Reboot the system:

```bash
sudo reboot
```

### Security Enhancements

Configure UFW (Uncomplicated Firewall) to enhance system security:

1. Install UFW:

```bash
sudo apt install ufw
```

2. Set default policies:

```bash
sudo ufw default deny incoming
sudo ufw default allow outgoing
```

3. Allow SSH access:

```bash
sudo ufw allow 22/tcp
```

4. Allow VNC access (if needed):

```bash
sudo ufw allow 5900/tcp
```

5. Enable the firewall:

```bash
sudo ufw enable
```

6. Check the status:

```bash
sudo ufw status verbose
```

## Remote Access

### SSH Configuration

SSH is enabled by default. Ensure you change the default password for security reasons.

### VNC Server Setup

Install and configure x11vnc for remote desktop access:

1. Install x11vnc:

```bash
sudo apt-get install x11vnc
```

2. Set VNC password:

```bash
sudo x11vnc -storepasswd /etc/x11vnc.pwd
```

3. Create a systemd service file:

```bash
sudo nano /lib/systemd/system/x11vnc.service
```

4. Add the following content:

```ini
[Unit]
Description=Start x11vnc at startup.
After=multi-user.target

[Service]
Type=simple
ExecStart=/usr/bin/x11vnc -display :0 -auth /home/pi/.Xauthority -forever -loop -noxdamage -repeat -rfbauth /etc/x11vnc.pwd -rfbport 5900 -shared -listen 0.0.0.0 -verbose
Restart=on-failure
RestartSec=2

[Install]
WantedBy=multi-user.target
```

5. Enable and start the x11vnc service:

```bash
sudo systemctl daemon-reload
sudo systemctl enable x11vnc.service
sudo systemctl start x11vnc
```

### Tailscale VPN Installation

Install Tailscale for secure remote access:

1. Install Tailscale:

```bash
curl -fsSL https://tailscale.com/install.sh | sh
```

2. Start Tailscale and authenticate:

```bash
sudo tailscale up
```

3. Follow the provided link to authenticate your device with your Tailscale account.

## NAS Functionality

### CasaOS Installation

CasaOS is a user-friendly, open-source home cloud system that enhances CM3588 NAS capabilities:

1. Install CasaOS:

```bash
curl -fsSL https://get.casaos.io | sudo bash
```

2. Access the CasaOS web interface by navigating to `http://your-cm3588-ip:80` in a web browser.

3. Follow the on-screen instructions to complete the CasaOS setup.

### HTTPS Configuration with Tailscale and Caddy

Secure your CasaOS interface with HTTPS using Tailscale's certificates and Caddy as a reverse proxy:

1. Install Caddy:

```bash
sudo apt install -y debian-keyring debian-archive-keyring apt-transport-https
curl -1sLf 'https://dl.cloudsmith.io/public/caddy/stable/gpg.key' | sudo gpg --dearmor -o /usr/share/keyrings/caddy-stable-archive-keyring.gpg
curl -1sLf 'https://dl.cloudsmith.io/public/caddy/stable/debian.deb.txt' | sudo tee /etc/apt/sources.list.d/caddy-stable.list
sudo apt update
sudo apt install caddy
```

2. Change CasaOS to listen on port 81:

```bash
sudo sed -i 's/80/81/g' /etc/casaos/gateway.ini
sudo systemctl restart casaos-gateway
```

3. Generate a Tailscale certificate:

```bash
tailscale cert your-device-name.your-tailnet.ts.net
```

4. Configure Caddy:

```bash
sudo nano /etc/caddy/Caddyfile
```

Add the following content:

```
{
    auto_https off
}

:443 {
    tls /etc/caddy/ssl/your-device-name.your-tailnet.ts.net.crt /etc/caddy/ssl/your-device-name.your-tailnet.ts.net.key
    reverse_proxy 127.0.0.1:81
}
```

5. Set correct permissions for certificate files:

```bash
sudo chown caddy:caddy /etc/caddy/ssl/your-device-name.your-tailnet.ts.net.crt /etc/caddy/ssl/your-device-name.your-tailnet.ts.net.key
sudo chmod 600 /etc/caddy/ssl/your-device-name.your-tailnet.ts.net.key
```

6. Configure Tailscale to allow Caddy to use the certificate:

```bash
echo "TS_PERMIT_CERT_UID=caddy" | sudo tee -a /etc/default/tailscaled
```

7. Restart Tailscale and Caddy:

```bash
sudo systemctl restart tailscaled
sudo systemctl restart caddy
```

You can now access your CasaOS interface securely via HTTPS using your Tailscale hostname:

```
https://your-device-name.your-tailnet.ts.net
```

## Advanced Topics

### Custom DNS Configuration

If using custom DNS services, ensure they're configured to properly resolve Tailscale's .ts.net domains. In some cases, using your system's default DNS settings may be necessary for proper resolution of Tailscale domains.

## Troubleshooting

For common issues and their solutions, refer to the [FriendlyElec CM3588 Wiki Troubleshooting section](https://wiki.friendlyelec.com/wiki/index.php/CM3588#Troubleshooting).

## Additional Resources

- [Official FriendlyElec CM3588 Documentation](https://wiki.friendlyelec.com/wiki/index.php/CM3588)
- [FriendlyElec Forum - CM3588 Section](https://www.friendlyelec.com/Forum/viewforum.php?f=71)
- [Tailscale Documentation](https://tailscale.com/kb/)
- [CasaOS Documentation](https://casaos.io/docs/)

Remember to regularly update your system and renew your Tailscale certificate every 90 days by re-running the `tailscale cert` command.



