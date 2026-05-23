# Initial Server Setup

Your first steps after getting a new VPS or EC2 server. Follow these steps in order.

---

## Step 1 — Connect to Your Server

**VPS (you get a root password):**
```bash
ssh root@YOUR_SERVER_IP
```

**AWS EC2 (you get a .pem key file):**
```bash
ssh -i /path/to/your-key.pem ubuntu@YOUR_SERVER_IP
```

> If you get a "permission denied" error on the .pem file, run:
> ```bash
> chmod 400 /path/to/your-key.pem
> ```

---

## Step 2 — Update the System

Always do this first. It installs the latest security patches.

```bash
sudo apt update && sudo apt upgrade -y
```

---

## Step 3 — Set Your Timezone

```bash
# See all timezones
timedatectl list-timezones | grep Asia

# Set your timezone (change to yours)
sudo timedatectl set-timezone Asia/Dhaka

# Check it worked
timedatectl
```

---

## Step 4 — Create a Normal User (Don't Use Root)

Using root every day is risky. Create a regular user instead.

```bash
# Create a new user (change "deployer" to any name you like)
sudo adduser deployer

# Give it admin (sudo) access
sudo usermod -aG sudo deployer
```

Now set up SSH key login for this user:

```bash
# Switch to the new user
su - deployer

# Create SSH folder
mkdir -p ~/.ssh
chmod 700 ~/.ssh

# Open the file where you'll paste your public key
nano ~/.ssh/authorized_keys
```

Paste your **public key** here. To find your public key on your local computer:
```bash
cat ~/.ssh/id_ed25519.pub
# or
cat ~/.ssh/id_rsa.pub
```

After pasting, save the file and set permissions:
```bash
chmod 600 ~/.ssh/authorized_keys
```

**Test it works before closing your current session:**
```bash
# Open a NEW terminal window and try:
ssh deployer@YOUR_SERVER_IP
```

---

## Step 5 — Secure SSH (Lock Down Login)

Edit the SSH settings file:

```bash
sudo nano /etc/ssh/sshd_config
```

Find and change these lines (add them if they don't exist):

```
Port 2222
PermitRootLogin no
PasswordAuthentication no
PubkeyAuthentication yes
AllowUsers deployer
MaxAuthTries 3
PermitEmptyPasswords no
X11Forwarding no
```

> **What each setting does:**
> - `Port 2222` — Changes SSH from the default port 22 (harder for bots to find)
> - `PermitRootLogin no` — Nobody can log in as root
> - `PasswordAuthentication no` — Only SSH keys work, no passwords
> - `AllowUsers deployer` — Only your user can log in

Save and restart SSH:
```bash
sudo systemctl restart sshd
```

> ⚠️ **Important:** Keep your current SSH session open. Open a new terminal and test login with port 2222 before closing anything!
> ```bash
> ssh -p 2222 deployer@YOUR_SERVER_IP
> ```

---

## Step 6 — Install Useful Tools

```bash
sudo apt install -y curl wget git htop nano unzip net-tools
```

---

## Step 7 — Create Swap Space (Extra Memory)

Useful for servers with 1-2 GB RAM. Swap is like extra memory on disk.

```bash
# Check if you already have swap
free -h

# Create a 2GB swap file
sudo fallocate -l 2G /swapfile
sudo chmod 600 /swapfile
sudo mkswap /swapfile
sudo swapon /swapfile

# Make it permanent (survives reboots)
echo '/swapfile none swap sw 0 0' | sudo tee -a /etc/fstab

# Reduce swap usage (use RAM first)
echo 'vm.swappiness=10' | sudo tee -a /etc/sysctl.conf
sudo sysctl -p
```

---

## Step 8 — Turn On Automatic Security Updates

This keeps your server automatically patched against security holes.

```bash
sudo apt install unattended-upgrades -y
sudo dpkg-reconfigure -plow unattended-upgrades
```

Select **Yes** when asked.

---

## Quick Checklist

- [ ] Connected to server
- [ ] System updated
- [ ] Timezone set
- [ ] New user created with sudo
- [ ] SSH key login working for new user
- [ ] Root login disabled
- [ ] Password login disabled
- [ ] SSH port changed to 2222
- [ ] Swap created (if needed)
- [ ] Auto security updates on

---

## Next Steps

1. [Firewall Setup](./02-firewall-configuration.md) — Block unwanted traffic
2. [SSH Security](./03-ssh-keys-and-security.md) — Protect against brute force attacks
3. [Install Nginx](./nginx-complete-guide.md) — Set up your web server
