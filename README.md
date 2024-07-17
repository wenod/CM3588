# FriendlyElec CM3588 SBC with NAS Kit - Setup Guide

This guide covers the setup process for the FriendlyElec CM3588 Single Board Computer (SBC) with NAS Kit. It includes steps for installing Debian Desktop (Bullseye), configuring essential services, and optimizing the system for use as a Network Attached Storage (NAS) device.
![image](https://github.com/user-attachments/assets/3b0c5de8-405d-4200-b5da-cfa08722fcac)

## Table of Contents

1. [Initial Setup](#initial-setup)
2. [System Configuration](#system-configuration)
3. [Network Configuration](#network-configuration)
4. [Remote Access](#remote-access)
5. [Localization](#localization)
6. [Security](#security)
7. [Additional Software](#additional-software)
8. [CasaOS Installation](#casaos-installation)
9. [Troubleshooting](#troubleshooting)

## Initial Setup

1. Download the latest Debian Desktop (Bullseye) image for CM3588 from the FriendlyElec website.
2. Use BalenaEtcher to create a bootable SD card with the image.
3. Insert the SD card into the CM3588 and power on the device.
4. Wait for the system to complete the initial boot and flash the eMMC.
5. Remove the SD card and reboot the device.

## System Configuration

### Update Package Sources

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

### Install Kernel Headers

```bash
sudo -i
dpkg -i /opt/archives/linux-headers-6.1.57_6.1.57-13_arm64.deb
```

### Install Proprietary Firmware (USB Wi-Fi Driver)

```bash
sudo apt update
sudo apt install firmware-realtek
sudo apt upgrade firmware-realtek
sudo modprobe -r r8188eu
sudo modprobe r8188eu
dmesg | grep r8188eu
sudo reboot
```

## Network Configuration

### Configure Wi-Fi

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

## Remote Access

### Install and Configure x11vnc Server

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

## Localization

### Set Timezone

1. View current timezone:

```bash
timedatectl
```

2. List available timezones:

```bash
timedatectl list-timezones
```

3. Set the desired timezone:

```bash
sudo timedatectl set-timezone Your/Timezone
```

### Configure Locale

1. Reconfigure locales:

```bash
sudo dpkg-reconfigure locales
```

2. Add locale environment variables to `.bashrc`:

```bash
echo "export LC_ALL=en_US.UTF-8" >> ~/.bashrc
echo "export LANG=en_US.UTF-8" >> ~/.bashrc
echo "export LANGUAGE=en_US.UTF-8" >> ~/.bashrc
```

3. Reboot the system:

```bash
sudo reboot
```

## Security

### Configure UFW (Uncomplicated Firewall)

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

## Additional Software

### Install Tailscale VPN

```bash
curl -fsSL https://tailscale.com/install.sh | sh
```

## CasaOS Installation

CasaOS is a user-friendly, open-source home cloud system that can enhance your CM3588 NAS capabilities.

1. Install CasaOS:

```bash
curl -fsSL https://get.casaos.io | sudo bash
```

2. Access the CasaOS web interface by navigating to `http://your-cm3588-ip:80` in a web browser.

3. Follow the on-screen instructions to complete the CasaOS setup.

## Troubleshooting

### CPU Fan Issues

If you experience problems with the CPU fan, refer to the FriendlyElec forum thread for potential solutions:
[CPU - GPIO CPU Fan Fix](https://www.friendlyelec.com/Forum/viewtopic.php?p=14618#p14300)

For more detailed information on the CM3588 and its features, consult the [official FriendlyElec CM3588 documentation](https://wiki.friendlyelec.com/wiki/index.php/CM3588).
