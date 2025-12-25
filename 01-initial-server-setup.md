# Initial VPS/EC2 Server Setup Guide

Complete guide for first-time server access, security hardening, and user management.

---

## Prerequisites

- A freshly provisioned VPS or AWS EC2 instance
- Root access or sudo privileges
- SSH client (Terminal, PuTTY, or similar)

---

## 1. First-Time SSH Connection

### For VPS (root access)

```bash
ssh root@your-server-ip
```

### For AWS EC2 (key-based)

```bash
ssh -i /path/to/your-key.pem ubuntu@your-ec2-ip

# For Amazon Linux
ssh -i /path/to/your-key.pem ec2-user@your-ec2-ip
```

> **Tip:** Set proper permissions for your key file:
> ```bash
> chmod 400 /path/to/your-key.pem
> ```

---

## 2. Update System Packages

Always update your system first:

```bash
# Debian/Ubuntu
sudo apt update && sudo apt upgrade -y

# CentOS/RHEL/Amazon Linux
sudo yum update -y

# Amazon Linux 2023
sudo dnf update -y
```

---

## 3. Set Hostname

```bash
# Set a meaningful hostname
sudo hostnamectl set-hostname your-server-name

# Verify
hostnamectl
```

Update `/etc/hosts`:

```bash
sudo nano /etc/hosts
```

Add:
```
127.0.0.1   your-server-name
```

---

## 4. Set Timezone

```bash
# List available timezones
timedatectl list-timezones

# Set your timezone
sudo timedatectl set-timezone Asia/Dhaka

# Verify
timedatectl
```

---

## 5. Create Non-Root User (Essential for Security)

Running as root is dangerous. Create a regular user:

```bash
# Create new user
sudo adduser deployer

# Add to sudo group
sudo usermod -aG sudo deployer
```

### Set Up SSH Keys for New User

```bash
# Switch to new user
su - deployer

# Create SSH directory
mkdir -p ~/.ssh
chmod 700 ~/.ssh

# Add your public key
nano ~/.ssh/authorized_keys
# Paste your public key (from your local machine: cat ~/.ssh/id_rsa.pub)

chmod 600 ~/.ssh/authorized_keys
```

Test login with new user before disabling root:

```bash
ssh deployer@your-server-ip
```

---

## 6. Secure SSH Configuration

Edit SSH config:

```bash
sudo nano /etc/ssh/sshd_config
```

Recommended settings:

```bash
# Disable root login
PermitRootLogin no

# Disable password authentication (after setting up SSH keys)
PasswordAuthentication no

# Change default port (optional but recommended)
Port 2222

# Allow only specific users
AllowUsers deployer

# Limit authentication attempts
MaxAuthTries 3

# Disable empty passwords
PermitEmptyPasswords no

# Enable public key authentication
PubkeyAuthentication yes

# Disable X11 forwarding
X11Forwarding no
```

Restart SSH:

```bash
sudo systemctl restart sshd
```

> ⚠️ **Warning:** Keep your current session open and test new connection in another terminal before closing!

---

## 7. Install Essential Packages

```bash
sudo apt install -y \
    curl \
    wget \
    git \
    htop \
    vim \
    nano \
    unzip \
    zip \
    tree \
    net-tools \
    software-properties-common \
    apt-transport-https \
    ca-certificates \
    gnupg \
    lsb-release
```

---

## 8. Configure Automatic Security Updates

```bash
sudo apt install unattended-upgrades -y
sudo dpkg-reconfigure -plow unattended-upgrades
```

Check configuration:

```bash
sudo nano /etc/apt/apt.conf.d/50unattended-upgrades
```

Enable security updates:

```
Unattended-Upgrade::Allowed-Origins {
    "${distro_id}:${distro_codename}-security";
};
```

---

## 9. Set Up Swap Space (If Needed)

Check current swap:

```bash
free -h
swapon --show
```

Create swap (recommended: 1-2x RAM for small servers):

```bash
# Create 2GB swap file
sudo fallocate -l 2G /swapfile
sudo chmod 600 /swapfile
sudo mkswap /swapfile
sudo swapon /swapfile

# Make permanent
echo '/swapfile none swap sw 0 0' | sudo tee -a /etc/fstab

# Verify
free -h
```

Adjust swappiness (lower = less swap usage):

```bash
sudo sysctl vm.swappiness=10
echo 'vm.swappiness=10' | sudo tee -a /etc/sysctl.conf
```

---

## 10. Basic System Information Commands

```bash
# System info
uname -a
lsb_release -a

# CPU info
nproc
lscpu

# Memory
free -h

# Disk space
df -h

# Network interfaces
ip a

# Running processes
htop
```

---

## Quick Checklist

- [ ] SSH connection working
- [ ] System updated
- [ ] Hostname set
- [ ] Timezone configured
- [ ] Non-root user created with sudo access
- [ ] SSH keys configured for new user
- [ ] Root login disabled
- [ ] Password authentication disabled
- [ ] Essential packages installed
- [ ] Automatic security updates enabled
- [ ] Swap space configured (if needed)

---

## Next Steps

1. **[Firewall Configuration](./02-firewall-configuration.md)** - Set up UFW/iptables
2. **[SSH Security](./03-ssh-keys-and-security.md)** - Advanced SSH hardening
3. **[Web Server Setup](./nginx-complete-guide.md)** - Install Nginx/Apache

---

✅ Your server is now ready for the next steps!
