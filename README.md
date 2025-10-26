# systemd-nspawn-distros

Multi-distro systemd-nspawn container guides for Arch Linux. Lightweight, fast, and easy to manage.

---

# Kali Linux systemd-nspawn Container Setup Guide

A complete guide to set up and manage Kali Linux containers using systemd-nspawn on Arch Linux.

## Table of Contents

- [Prerequisites](#prerequisites)
- [Installation](#installation)
- [Initial Container Setup](#initial-container-setup)
- [Container Management with SNR](#container-management-with-snr)
- [Multiple Terminal Sessions](#multiple-terminal-sessions)
- [Install Full Kali Linux](#install-full-kali-linux-optional)
- [Troubleshooting](#troubleshooting)
- [Useful Commands](#useful-commands)
- [Advanced Configuration](#advanced-configuration)
- [Security Considerations](#security-considerations)

---

## Prerequisites

- Arch Linux or Arch-based distribution
- Root or sudo access
- Basic knowledge of Linux terminal

---

## Installation

### Step 1: Install debootstrap

```bash
sudo pacman -Syu debootstrap
```

### Step 2: Create Container Directory

```bash
# Navigate to containers directory
cd /var/lib/containers

# Or create your own directory
sudo mkdir -p /path/of/container
cd /path/of/container
```

### Step 3: Bootstrap Kali Linux

This will download and install a minimal Kali Linux system:

```bash
sudo debootstrap \
  --include=dbus-broker,systemd-container \
  --components=main,non-free,contrib,non-free-firmware \
  kali-rolling \
  kali \
  http://http.kali.org/kali
```

**This process may take several minutes depending on your internet connection.**

---

## Initial Container Setup

### Step 1: Start the Container (First Time)

```bash
sudo systemd-nspawn -D /var/lib/containers/kali
```

### Step 2: Initialize the Container

Once inside the container, perform these initial configurations:

#### Set Root Password

```bash
passwd
```

#### Install and Configure Locales

```bash
# Install locales package
apt install locales

# Edit locale configuration
nano /etc/locale.gen
```

Uncomment your desired locale (e.g., `en_US.UTF-8 UTF-8`), then:

```bash
# Generate locales
locale-gen
```

### Step 3: Exit the Container

Press `Ctrl+]` three times within 1 second to kill the container.

---

## Container Management with SNR

SNR (SystemD-NsPawn Runner) is a convenient wrapper for managing systemd-nspawn containers.

### Install SNR

```bash
# Create temporary directory
mkdir -p ~/snr-tmp
cd ~/snr-tmp

# Clone the repository
git clone https://github.com/mikhailnov/snr.git

# Copy files to system directories
sudo cp snr/snr.sh /usr/bin/snr
sudo cp snr/snr.conf /etc/snr.conf

# Make executable
sudo chmod +x /usr/bin/snr
```

### Configure SNR

Edit the SNR configuration file:

```bash
sudo nano /etc/snr.conf
```

Add your container configuration:

```bash
# Container directory path
DIR="/var/lib/containers"

# Additional systemd-nspawn options
other_options="--boot --capability=CAP_NET_ADMIN --background="
```

**Options explained:**

- `--boot`: Boot the container with systemd init
- `--capability=CAP_NET_ADMIN`: Allow network administration
- `--background=""`: Disable terminal background tinting (systemd 256+)

### Start Container with SNR

```bash
snr kali
```

**Login credentials:**

- Username: `root`
- Password: (the password you set earlier)

---

## Multiple Terminal Sessions

You can open multiple terminal sessions to the same running container without creating new instances.

### Method 1: machinectl shell (Recommended)

```bash
# Open a new shell in the running container
sudo machinectl shell kali

# Or specify a user
sudo machinectl shell root@kali
```

### Method 2: systemd-run

```bash
sudo systemd-run -M kali -t /bin/bash
```

### Method 3: machinectl login

```bash
sudo machinectl login kali
```

---

## Install Full Kali Linux (Optional)

Once inside the container, you can install the full Kali Linux package set:

```bash
# Update package lists
apt update

# Upgrade existing packages
apt upgrade

# Install default Kali Linux metapackage
apt install -y kali-linux-default

# Or install the large metapackage (includes more tools)
# apt install -y kali-linux-large
```

---

## Troubleshooting

### Issue: `xhost: command not found`

This is a warning when starting the container with SNR. It's not critical unless you need GUI applications.

**Solution (if you need GUI support):**

```bash
# On host system
sudo pacman -S xorg-xhost
```

**Alternative: Disable xhost in SNR**

```bash
sudo nano /usr/bin/snr
```

Find line 161 and comment it out:

```bash
# xhost +local:
```

### Issue: Blue Terminal Background (systemd 256+)

This was a feature introduced in systemd 256 that tints container terminals.

**Solution:** Use `--background=""` option (already included in the configuration above).

### Issue: Container Won't Start

Check if the container is already running:

```bash
sudo machinectl list
```

If running, stop it first:

```bash
sudo machinectl poweroff kali
```

---

## Useful Commands

### Container Status

```bash
# List running containers
sudo machinectl list

# Check container status
sudo machinectl status kali
```

### Container Control

```bash
# Start container
snr kali

# Stop container
sudo machinectl poweroff kali

# Reboot container
sudo machinectl reboot kali

# Kill container (force stop)
sudo machinectl terminate kali
```

### Container Shell Access

```bash
# Interactive shell
sudo machinectl shell kali

# Run single command
sudo machinectl shell kali /usr/bin/command

# Login to container
sudo machinectl login kali
```

### Container Information

```bash
# Show container properties
sudo machinectl show kali

# Show container resource usage
sudo systemd-cgtop

# View container logs
sudo journalctl -M kali
```

---

## Advanced Configuration

### Networking

SNR automatically configures networking with `--capability=CAP_NET_ADMIN`. For manual network setup:

```bash
# Inside the container
apt install iproute2 iputils-ping

# Check network interfaces
ip addr show

# Configure network (if needed)
nano /etc/network/interfaces
```

### Bind Mounts

To share directories between host and container, modify SNR configuration:

```bash
sudo nano /etc/snr.conf
```

Add bind mounts:

```bash
other_options="--boot --capability=CAP_NET_ADMIN --background= --bind=/home/user/shared:/mnt/shared"
```

### Enable systemd Services

Inside the container:

```bash
# Enable SSH server
systemctl enable ssh
systemctl start ssh

# Enable other services as needed
systemctl enable <service-name>
```

---

## Security Considerations

1. **Container Isolation**: systemd-nspawn provides process and filesystem isolation but shares the kernel with the host.
2. **Root Access**: The container root user is separate from the host root user.
3. **Capabilities**: Only grant necessary capabilities (like `CAP_NET_ADMIN`) to containers.
4. **Updates**: Keep both host and container systems updated regularly.

---

## Resources

- [systemd-nspawn Documentation](https://www.freedesktop.org/software/systemd/man/systemd-nspawn.html)
- [SNR GitHub Repository](https://github.com/mikhailnov/snr)
- [Kali Linux Official Documentation](https://www.kali.org/docs/)
- [Arch Linux Wiki - systemd-nspawn](https://wiki.archlinux.org/title/Systemd-nspawn)

---

## Contributing

Found an issue or want to improve this guide? Feel free to submit a pull request or open an issue.

## License

This guide is provided as-is for educational purposes. Use at your own risk.

---

**Last Updated:** October 2025
