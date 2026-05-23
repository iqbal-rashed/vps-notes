# SSH Keys and Brute Force Protection

This guide covers two things:
1. **SSH Keys** — How to set up key-based login (more secure than passwords)
2. **Fail2Ban** — Automatically blocks IPs that try to hack into your server

---

## Part 1 — SSH Keys

### Create an SSH Key (on your LOCAL computer)

Open a terminal on your **local machine** (not the server):

```bash
ssh-keygen -t ed25519 -C "your-email@example.com"
```

- Press Enter to save in the default location
- Add a passphrase for extra security (recommended)

This creates two files:
- `~/.ssh/id_ed25519` — **Private key** (keep this secret, never share)
- `~/.ssh/id_ed25519.pub` — **Public key** (this goes on your server)

---

### Copy Your Key to the Server

**Easy way (if password login is still on):**
```bash
ssh-copy-id -i ~/.ssh/id_ed25519.pub deployer@YOUR_SERVER_IP
```

**Manual way:**
1. On your local machine, copy your public key:
```bash
cat ~/.ssh/id_ed25519.pub
```

2. On the server, paste it:
```bash
mkdir -p ~/.ssh
chmod 700 ~/.ssh
nano ~/.ssh/authorized_keys
# Paste your public key on a new line
chmod 600 ~/.ssh/authorized_keys
```

---

### Make SSH Connection Easier (Optional)

Create a config file on your **local machine** so you don't have to type the full command every time:

```bash
nano ~/.ssh/config
```

```
Host myserver
    HostName YOUR_SERVER_IP
    User deployer
    Port 2222
    IdentityFile ~/.ssh/id_ed25519
```

Now you can connect with just:
```bash
ssh myserver
```

---

## Part 2 — Fail2Ban (Block Attackers Automatically)

Fail2Ban watches your server logs. If someone tries to log in and fails too many times, it bans their IP address.

### Install Fail2Ban

```bash
sudo apt install fail2ban -y
```

### Configure Fail2Ban

Create a local config file (don't edit the main one):

```bash
sudo cp /etc/fail2ban/jail.conf /etc/fail2ban/jail.local
sudo nano /etc/fail2ban/jail.local
```

Find or add these settings:

```ini
[DEFAULT]
# Ban the IP for 1 hour
bantime = 1h

# How many failures before ban
maxretry = 5

# Time window to count failures
findtime = 10m

# Don't ban your own IP
ignoreip = 127.0.0.1/8 ::1

[sshd]
enabled = true
# Change to 2222 if you changed your SSH port
port = 2222
maxretry = 3
bantime = 24h
```

### Start Fail2Ban

```bash
sudo systemctl enable fail2ban
sudo systemctl start fail2ban
```

### Check Fail2Ban Status

```bash
# See overview
sudo fail2ban-client status

# See SSH bans specifically
sudo fail2ban-client status sshd

# Unban an IP (if you accidentally banned yourself)
sudo fail2ban-client set sshd unbanip YOUR_IP_ADDRESS

# See the log
sudo tail -f /var/log/fail2ban.log
```

---

## Security Checklist

- [ ] SSH key generated on local machine
- [ ] Public key added to server
- [ ] SSH password login disabled (`PasswordAuthentication no`)
- [ ] SSH root login disabled (`PermitRootLogin no`)
- [ ] SSH port changed (default 22 → 2222)
- [ ] Fail2Ban installed and running
- [ ] Firewall enabled (see [Firewall Guide](./02-firewall-configuration.md))

---

## Troubleshooting

**"Permission denied (publickey)" error:**
```bash
# Fix SSH folder permissions on the server
chmod 700 ~/.ssh
chmod 600 ~/.ssh/authorized_keys

# Try connecting with debug info to see what's wrong
ssh -v deployer@YOUR_SERVER_IP
```

**"Connection refused" error:**
```bash
# Make sure SSH is running on the server
sudo systemctl status sshd

# Check the port is correct
sudo ss -tlnp | grep ssh

# Check firewall allows the port
sudo ufw status
```

---

## Next Steps

1. [Firewall Setup](./02-firewall-configuration.md) — Block unwanted traffic
2. [Nginx Setup](./nginx-complete-guide.md) — Install web server
3. [SSL/HTTPS](./certbot-installation.md) — Free SSL certificate
