# Arch Linux systemd-nspawn Container Setup Guide

## From a Debian-based Host

A guide to set up and manage an Arch Linux container using systemd-nspawn from a **Debian-based host** (like Debian, Ubuntu, or Kali Linux).

## Table of Contents

- [Prerequisites](#prerequisites)
- [Installation](#installation)
- [Initial Container Setup](#initial-container-setup)
- [Container Management](#container-management)

---

## Prerequisites

- A Debian-based distribution (e.g., Debian, Ubuntu, Kali)
- Root or sudo access
- `wget` and `zstd` installed (`sudo apt install wget zstd`)
- Basic knowledge of the Linux terminal

---

## Installation

Setting up Arch Linux on a non-Arch host requires using a bootstrap tarball, as `pacstrap` is not available.

### Step 1: Download the Arch Linux Bootstrap Tarball

1.  Find the latest bootstrap tarball URL from the [Arch Linux download page](https://archlinux.org/download/) (under "Netboot"). The filename will be similar to `archlinux-bootstrap-x86_64.tar.zst`.

2.  Download the tarball and its signature:

    ```bash
    # The version number changes, so update the URL accordingly
    https://geo.mirror.pkgbuild.com/iso/latest/
    ```

#### Example:

```bash
    wget https://geo.mirror.pkgbuild.com/iso/latest/archlinux-bootstrap-2025.11.01-x86_64.tar.zst
    wget https://geo.mirror.pkgbuild.com/iso/latest/archlinux-bootstrap-2025.11.01-x86_64.tar.zst.sig
```

### Step 2: Verify the Tarball (Recommended)

1.  Import the signing key for the developer who signed the image. The key ID is typically found on the download page or related news posts. For this example, we'll assume a key.

    ```bash
    # This key is an EXAMPLE. Find the correct one from Arch's website.
    gpg --keyserver-options auto-key-retrieve --verify archlinux-bootstrap-2025.11.01-x86_64.tar.zst.sig
    ```

    If the verification is successful, you can proceed.

### Step 3: Create and Extract to the Container Directory

1.  Create the root directory for your container. A common location is `/var/lib/containers`.

    ```bash
    sudo mkdir -p /var/lib/containers/arch
    ```

2.  Extract the tarball into the new directory.

    ```bash
    sudo tar -C /var/lib/containers/arch -xaf archlinux-bootstrap-*.tar.zst
    ```

    This will create a `root.x86_64` directory inside `/var/lib/containers/arch`. We need to move the contents up one level.

3.  Move the contents to the correct root.
    ```bash
    sudo mv /var/lib/containers/arch/root.x86_64/* /var/lib/containers/arch/
    sudo rmdir /var/lib/containers/arch/root.x86_64
    ```

---

## Initial Container Setup

### Step 1: Configure DNS Resolution

The bootstrap environment doesn't have DNS configured. Copy your host's `resolv.conf` so `pacman` can reach the mirrors.

```bash
sudo cp /etc/resolv.conf /var/lib/containers/arch/etc/
```

### Step 2: Enter the Container and Initialize Pacman

Use `systemd-nspawn` to chroot into the new environment and set up `pacman`.

```bash
sudo systemd-nspawn -D /var/lib/containers/arch

# Inside the container, run the following commands:
# The bootstrap image has its own pacman-key, which we must initialize.
pacman-key --init
pacman-key --populate archlinux
```

### Step 3: Install Base System & Configure

Now, install the `base` package and other essentials.

```bash
# Still inside the container...

# It's a good idea to update the mirrorlist first. You can use a text editor
# or just proceed. For simplicity, we'll proceed.
pacman -Syu --noconfirm base haveged

# Set a root password
passwd

# Generate locales
echo "en_US.UTF-8 UTF-8" > /etc/locale.gen
locale-gen
echo "LANG=en_US.UTF-8" > /etc/locale.conf

# Exit the container
exit
```

Press `Ctrl+]` three times if `exit` doesn't work.

---

## Container Management

### Starting and Stopping

You can now boot the full container. `systemd-nspawn` will automatically manage services like `haveged` to ensure entropy for key generation.

```bash
# Boot the container with systemd
sudo systemd-nspawn -bD /var/lib/containers/arch
```

- `-b` boots the container's own init system.
- `-D` specifies the container directory.

Log in with `root` and the password you set.

To stop the container:

```bash
sudo machinectl poweroff arch
```

### Getting a Shell

To get a shell in a running container:

```bash
sudo machinectl shell arch
```

For more advanced management, you can use tools like `snr` as described in the other guides, but you would need to install `git` and configure it first inside your new Arch container.
