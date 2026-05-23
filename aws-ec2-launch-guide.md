# AWS EC2 Setup Guide

Step-by-step guide to create and connect to an AWS EC2 server.

---

## Step 1 — Create an EC2 Instance

1. Log into [AWS Console](https://console.aws.amazon.com)
2. Go to **EC2** → click **Launch Instance**

Fill in these settings:

**Name:** `my-web-server` (or anything you like)

**OS:** Ubuntu Server 24.04 LTS (recommended for most apps)

**Instance Type:**

| Type | RAM | Good For |
|------|-----|----------|
| t2.micro | 1 GB | Testing (free tier) |
| t3.small | 2 GB | Small sites |
| t3.medium | 4 GB | Production apps |
| t3.large | 8 GB | Busy apps |

**Key Pair:**
- Click **Create new key pair**
- Name it `my-server-key`
- Type: RSA, Format: `.pem`
- **Download it and keep it safe** — you can't download it again!

**Firewall (Security Group):**
- Allow SSH from **My IP** only (port 22)
- Allow HTTP from **Anywhere** (port 80)
- Allow HTTPS from **Anywhere** (port 443)

**Storage:** 20 GB, type `gp3` (SSD)

Click **Launch Instance**.

---

## Step 2 — Get a Static IP (Elastic IP)

By default, your server IP changes every time it restarts. Fix this with a free Elastic IP.

1. Go to **EC2** → **Elastic IPs** (left sidebar)
2. Click **Allocate Elastic IP address** → **Allocate**
3. Select the new IP → **Actions** → **Associate Elastic IP address**
4. Choose your instance → **Associate**

> Elastic IPs are free as long as they're attached to a running instance.

---

## Step 3 — Connect to Your Server

```bash
# Set the correct permissions on your key file
chmod 400 my-server-key.pem

# Connect
ssh -i my-server-key.pem ubuntu@YOUR_SERVER_IP
```

**Make it easier** — add this to `~/.ssh/config` on your local machine:

```
Host aws-server
    HostName YOUR_SERVER_IP
    User ubuntu
    IdentityFile ~/.ssh/my-server-key.pem
```

Now you can connect with just:
```bash
ssh aws-server
```

---

## Step 4 — First-Time Server Setup

After connecting, run these commands:

```bash
# Update the system
sudo apt update && sudo apt upgrade -y

# Set your timezone
sudo timedatectl set-timezone Asia/Dhaka

# Install useful tools
sudo apt install -y curl wget git htop nano unzip

# Create a regular user (don't use ubuntu/root for everything)
sudo adduser deployer
sudo usermod -aG sudo deployer
```

Then follow the [Initial Server Setup guide](./01-initial-server-setup.md) to finish securing it.

---

## Step 5 — Point Your Domain to This Server

In your domain's DNS settings, add these records:

| Type | Name | Value |
|------|------|-------|
| A | `@` | Your Elastic IP |
| A | `www` | Your Elastic IP |

DNS changes can take a few minutes to a few hours to work.

---

## Security Group Rules (Firewall)

AWS Security Groups are like a firewall. Keep these principles:

- **SSH (port 22)** — Only allow from **your own IP**, not the whole internet
- **HTTP/HTTPS (80/443)** — Allow from anywhere
- **Databases (3306, 5432, 27017, 6379)** — Never open to the public. Leave them blocked.

To update rules: EC2 → your instance → **Security** tab → click the security group → **Edit inbound rules**

---

## Useful Tips

**Check your server's public IP from inside the server:**
```bash
curl http://169.254.169.254/latest/meta-data/public-ipv4
```

**Add swap space (important for t2.micro / t3.small):**
```bash
sudo fallocate -l 2G /swapfile
sudo chmod 600 /swapfile
sudo mkswap /swapfile
sudo swapon /swapfile
echo '/swapfile none swap sw 0 0' | sudo tee -a /etc/fstab
```

**Stop your instance when not using it** — you pay per hour it's running.
Go to EC2 → select instance → **Instance State** → **Stop**.

---

## Troubleshooting

**Can't connect via SSH:**
- Check the Security Group has port 22 open for your IP
- Make sure the instance is in **Running** state
- Check you're using the right username (`ubuntu` for Ubuntu, `ec2-user` for Amazon Linux)
- Make sure the `.pem` file has `chmod 400` permissions

**Instance keeps stopping:**
- t2.micro can run out of CPU credits — upgrade to t3.micro or add swap

---

## Next Steps

1. [Initial Server Setup](./01-initial-server-setup.md) — Secure your server
2. [Firewall Setup](./02-firewall-configuration.md) — UFW on top of Security Groups
3. [Install Nginx](./nginx-complete-guide.md) — Set up web server
