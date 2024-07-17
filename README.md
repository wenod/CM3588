# FriendlyElec CM3588 SBC with NAS Kit: Debian Bullseye Installation and Configuration

This guide provides step-by-step instructions for setting up a FriendlyElec CM3588 SBC with NAS Kit, running Debian Bullseye Desktop. It covers essential configurations and optimizations for this specific board.

## Table of Contents
1. [Initial Setup](#initial-setup)
2. [System Configuration](#system-configuration)
3. [Network Configuration](#network-configuration)
4. [Remote Access Setup](#remote-access-setup)
5. [Additional Configurations](#additional-configurations)
6. [NAS Specific Setup](#nas-specific-setup)
7. [Troubleshooting](#troubleshooting)

## Initial Setup

### Creating a Bootable SD Card
1. Download the Debian-Desktop-Bullseye image for CM3588 SBC from the FriendlyElec website.
2. Use BalenaEtcher to create a bootable SD card with the downloaded image.

### Installation Process
1. Insert the SD card into the CM3588 SBC and power on the device.
2. Wait for the system to complete flashing to the eMMC storage.
3. Once finished, remove the SD card and reboot the device.
4. Use a network scanner (e.g., IP Scanner) to find the IP address of your CM3588 SBC.

## System Configuration

### Accessing the CM3588 SBC
1. Connect to the CM3588 SBC via SSH:
   ```
   ssh pi@<device_ip>
   ```
   Note: The default username is 'pi'. Make sure to change the password upon first login.

### Updating Package Sources
1. Update the package sources list:
   ```
   sudo mv /etc/apt/sources.list /etc/apt/sources.list.old
   sudo nano /etc/apt/sources.list
   ```
2. Add the following content to the new `sources.list` file:
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
3. Update the package list:
   ```
   sudo apt-get update
   ```

### Installing Kernel Headers
```
sudo -i
dpkg -i /opt/archives/linux-headers-6.1.57_6.1.57-13_arm64.deb
```

### Installing Proprietary Firmware (USB Wi-Fi Driver)
```
sudo apt update
sudo apt install firmware-realtek
sudo apt upgrade firmware-realtek
sudo modprobe -r r8188eu
sudo modprobe r8188eu
dmesg | grep r8188eu
sudo reboot
```

## Network Configuration

### Configuring Wi-Fi
1. List available Wi-Fi networks:
   ```
   nmcli device wifi list
   ```
2. Connect to a Wi-Fi network:
   ```
   sudo nmcli device wifi connect <SSID> password "<password>"
   ```
3. Set auto-connect for the Wi-Fi network:
   ```
   sudo nmcli connection modify <SSID> connection.autoconnect yes
   ```
4. Set connection priority (optional):
   ```
   sudo nmcli connection modify <SSID> connection.autoconnect-priority <priority_number>
   ```

## Remote Access Setup

### Installing and Configuring x11vnc Server
1. Install x11vnc:
   ```
   sudo apt-get install x11vnc
   ```
2. Set VNC password:
   ```
   sudo x11vnc -storepasswd /etc/x11vnc.pwd
   ```
3. Create a systemd service file:
   ```
   sudo nano /lib/systemd/system/x11vnc.service
   ```
4. Add the following content to the service file:
   ```
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
   ```
   sudo systemctl daemon-reload
   sudo systemctl enable x11vnc.service
   sudo systemctl start x11vnc
   ```

## Additional Configurations

### Setting the Timezone
```
timedatectl list-timezones
sudo timedatectl set-timezone <your_timezone>
```

### Configuring Locales
1. Reconfigure locales:
   ```
   sudo dpkg-reconfigure locales
   ```
2. Add locale settings to `.bashrc`:
   ```
   echo "export LC_ALL=en_US.UTF-8" >> ~/.bashrc
   echo "export LANG=en_US.UTF-8" >> ~/.bashrc
   echo "export LANGUAGE=en_US.UTF-8" >> ~/.bashrc
   ```
3. Reboot the system:
   ```
   sudo reboot
   ```

### Installing Tailscale (Optional)
For secure remote access, you can install Tailscale:
```
curl -fsSL https://tailscale.com/install.sh | sh
```

## NAS Specific Setup

The CM3588 SBC comes with a NAS kit, which allows you to use it as a Network Attached Storage device. Here are some steps to set up the NAS functionality:

1. Install necessary packages for file sharing:
   ```
   sudo apt install samba samba-common-bin
   ```

2. Configure Samba for file sharing:
   ```
   sudo nano /etc/samba/smb.conf
   ```
   Add your shared folder configuration at the end of the file.

3. Create a Samba user:
   ```
   sudo smbpasswd -a pi
   ```

4. Restart Samba service:
   ```
   sudo systemctl restart smbd
   ```

5. For advanced NAS features, consider installing and configuring OpenMediaVault, a dedicated NAS solution:
   ```
   wget -O - https://github.com/OpenMediaVault-Plugin-Developers/installScript/raw/master/install | sudo bash
   ```

## Troubleshooting

### CPU Fan Issues
If you encounter CPU fan problems, check the GPIO settings for the CM3588 SBC. You may need to adjust the fan control script based on the specific GPIO pins used in this board.

## Additional Resources
For more detailed information and advanced configurations, refer to the official FriendlyElec documentation for the CM3588 SBC with NAS Kit:
[FriendlyElec CM3588 SBC Documentation](https://wiki.friendlyelec.com/wiki/index.php/CM3588)
