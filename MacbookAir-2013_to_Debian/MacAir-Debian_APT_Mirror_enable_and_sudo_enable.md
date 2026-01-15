## Enable APT network mirror (replace CD-ROM source) + update/upgrade + install sudo

**Problem:** After an offline/minimal install, `apt` could not find packages (e.g., `Unable to locate package sudo`) because APT was only using the **CD-ROM (installer ISO)** as a source.

**Fix:** Replace the CD-ROM sources with Debian online repos (`deb.debian.org`), then update/upgrade and install `sudo`.

### 1) Become root

```bash
su -
```

### 2) Replace APT sources with `deb.debian.org`

Get your Debian codename (e.g., `bookworm`):

```bash
. /etc/os-release
echo "$VERSION_CODENAME"
```

Create/replace `/etc/apt/sources.list` (includes firmware repos for Broadcom Wi-Fi later):

```bash
. /etc/os-release
cat > /etc/apt/sources.list <<EOF
deb https://deb.debian.org/debian $VERSION_CODENAME main contrib non-free non-free-firmware
deb https://deb.debian.org/debian $VERSION_CODENAME-updates main contrib non-free non-free-firmware
deb https://security.debian.org/debian-security $VERSION_CODENAME-security main contrib non-free non-free-firmware
EOF
```

> Note: If your file had `deb cdrom:` entries, they were removed/replaced by the above.

### 3) Update package lists and upgrade

```bash
apt update
apt upgrade -y
```

### 4) Install sudo

```bash
apt install -y sudo
```

### 5) Add user to sudo group

Replace `YOURUSER` with your username:

```bash
usermod -aG sudo YOURUSER
```

### 6) Apply group changes

Log out and back in (or reboot):

```bash
reboot
```

### 7) Verify sudo works

After logging back in:

```bash
sudo -v
sudo whoami
```

Expected output: `root`