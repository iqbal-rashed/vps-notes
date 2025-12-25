# SSH Keys and Security Guide

Complete guide for SSH key management, fail2ban setup, and advanced security hardening.

---

## SSH Key Generation

### Generate Key Pair (Local Machine)

```bash
# Ed25519 (Recommended - more secure, shorter keys)
ssh-keygen -t ed25519 -C "your-email@example.com"

# RSA 4096-bit (for older systems)
ssh-keygen -t rsa -b 4096 -C "your-email@example.com"
```

Options:
- Press Enter to accept default location (`~/.ssh/id_ed25519`)
- Enter a passphrase (recommended for extra security)

### View Your Public Key

```bash
cat ~/.ssh/id_ed25519.pub
# or
cat ~/.ssh/id_rsa.pub
```

### Copy Key to Server

**Method 1: ssh-copy-id (Easiest)**

```bash
ssh-copy-id -i ~/.ssh/id_ed25519.pub user@server-ip
```

**Method 2: Manual Copy**

```bash
# On your local machine
cat ~/.ssh/id_ed25519.pub

# On server, paste into:
mkdir -p ~/.ssh
chmod 700 ~/.ssh
nano ~/.ssh/authorized_keys
# Paste your public key
chmod 600 ~/.ssh/authorized_keys
```

**Method 3: Single Command**

```bash
cat ~/.ssh/id_ed25519.pub | ssh user@server-ip "mkdir -p ~/.ssh && chmod 700 ~/.ssh && cat >> ~/.ssh/authorized_keys && chmod 600 ~/.ssh/authorized_keys"
```

---

## SSH Config File (Local Machine)

Simplify connections with `~/.ssh/config`:

```bash
nano ~/.ssh/config
```

```bash
# Production Server
Host prod
    HostName 192.168.1.100
    User deployer
    Port 2222
    IdentityFile ~/.ssh/id_ed25519
    
# Staging Server
Host staging
    HostName 192.168.1.101
    User deployer
    IdentityFile ~/.ssh/id_ed25519
    
# AWS EC2
Host aws-web
    HostName ec2-xx-xx-xx-xx.compute-1.amazonaws.com
    User ubuntu
    IdentityFile ~/.ssh/aws-key.pem
    
# Jump host / Bastion
Host production-db
    HostName 10.0.0.5
    User admin
    ProxyJump prod
    
# Default settings for all hosts
Host *
    ServerAliveInterval 60
    ServerAliveCountMax 3
    AddKeysToAgent yes
```

Now connect with:

```bash
ssh prod
ssh staging
ssh aws-web
```

---

## SSH Agent

Keep your keys loaded in memory:

### Start SSH Agent

```bash
# Start agent
eval "$(ssh-agent -s)"

# Add key
ssh-add ~/.ssh/id_ed25519

# List loaded keys
ssh-add -l

# Remove all keys
ssh-add -D
```

### Persistent Agent (Add to ~/.bashrc or ~/.zshrc)

```bash
# Auto-start SSH agent
if [ -z "$SSH_AUTH_SOCK" ]; then
   eval "$(ssh-agent -s)"
   ssh-add ~/.ssh/id_ed25519
fi
```

---

## SSH Agent Forwarding

Use your local keys on remote servers (useful for Git):

### Enable in SSH Config

```bash
Host prod
    HostName 192.168.1.100
    User deployer
    ForwardAgent yes
```

### Or use command flag

```bash
ssh -A user@server
```

### Verify on Remote Server

```bash
ssh-add -l
# Should show your local keys
```

> ⚠️ **Security Warning:** Only use agent forwarding on trusted servers!

---

## Fail2Ban Setup

Protect against brute force attacks.

### Install Fail2Ban

```bash
sudo apt install fail2ban -y
```

### Configure Fail2Ban

```bash
# Create local config (don't edit jail.conf directly)
sudo cp /etc/fail2ban/jail.conf /etc/fail2ban/jail.local
sudo nano /etc/fail2ban/jail.local
```

### Recommended Settings

```ini
[DEFAULT]
# Ban for 1 hour
bantime = 1h

# Number of failures before ban
maxretry = 5

# Detection window
findtime = 10m

# Email notifications (optional)
destemail = your-email@example.com
sender = fail2ban@your-server.com
mta = sendmail
action = %(action_mwl)s

# Ignore local IPs
ignoreip = 127.0.0.1/8 ::1

[sshd]
enabled = true
port = ssh
# Or custom port:
# port = 2222
filter = sshd
logpath = /var/log/auth.log
maxretry = 3
bantime = 24h
```

### Additional Jails

```ini
# Nginx authentication failures
[nginx-http-auth]
enabled = true
port = http,https
filter = nginx-http-auth
logpath = /var/log/nginx/error.log
maxretry = 3

# Nginx rate limiting
[nginx-limit-req]
enabled = true
port = http,https
filter = nginx-limit-req
logpath = /var/log/nginx/error.log
maxretry = 10
findtime = 1m
bantime = 10m

# Bad bots
[nginx-badbots]
enabled = true
port = http,https
filter = nginx-badbots
logpath = /var/log/nginx/access.log
maxretry = 2
bantime = 86400
```

### Manage Fail2Ban

```bash
# Restart service
sudo systemctl restart fail2ban

# Check status
sudo fail2ban-client status
sudo fail2ban-client status sshd

# Unban an IP
sudo fail2ban-client set sshd unbanip 192.168.1.100

# Ban an IP manually
sudo fail2ban-client set sshd banip 192.168.1.100

# Check banned IPs
sudo fail2ban-client get sshd banned

# View logs
sudo tail -f /var/log/fail2ban.log
```

---

## Advanced SSH Hardening

### Enhanced sshd_config

```bash
sudo nano /etc/ssh/sshd_config
```

```bash
# Basics
Port 2222
Protocol 2
HostKey /etc/ssh/ssh_host_ed25519_key
HostKey /etc/ssh/ssh_host_rsa_key

# Authentication
LoginGraceTime 30
PermitRootLogin no
StrictModes yes
MaxAuthTries 3
MaxSessions 3

# Key-based auth only
PubkeyAuthentication yes
PasswordAuthentication no
PermitEmptyPasswords no
ChallengeResponseAuthentication no

# Disable unused auth methods
KerberosAuthentication no
GSSAPIAuthentication no

# Disable tunneling (unless needed)
AllowTcpForwarding no
X11Forwarding no
AllowAgentForwarding no

# Restrict users
AllowUsers deployer admin
# Or restrict groups:
# AllowGroups sshusers

# Logging
SyslogFacility AUTH
LogLevel VERBOSE

# Keep connections alive
ClientAliveInterval 300
ClientAliveCountMax 2

# Security
UsePAM yes
UseDNS no

# Banner
Banner /etc/ssh/banner

# Restrict ciphers (modern only)
Ciphers chacha20-poly1305@openssh.com,aes256-gcm@openssh.com,aes128-gcm@openssh.com
MACs hmac-sha2-512-etm@openssh.com,hmac-sha2-256-etm@openssh.com
KexAlgorithms curve25519-sha256,curve25519-sha256@libssh.org
```

### Create Login Banner

```bash
sudo nano /etc/ssh/banner
```

```
*******************************************************************
*                   AUTHORIZED ACCESS ONLY                         *
* This system is for authorized use only.                         *
* All activities may be monitored and recorded.                   *
* Unauthorized access is strictly prohibited.                     *
*******************************************************************
```

### Apply Changes

```bash
# Test config first
sudo sshd -t

# Restart SSH
sudo systemctl restart sshd
```

---

## Two-Factor Authentication (2FA)

Add extra security with Google Authenticator.

### Install

```bash
sudo apt install libpam-google-authenticator -y
```

### Configure for User

```bash
# Run as the user (not root)
google-authenticator
```

Answer the prompts:
- Yes to time-based tokens
- Scan QR code with authenticator app
- Save emergency codes
- Yes to update .google_authenticator
- Yes to disallow multiple uses
- No to increasing time window (unless needed)
- Yes to rate limiting

### Enable in PAM

```bash
sudo nano /etc/pam.d/sshd
```

Add at the end:
```
auth required pam_google_authenticator.so
```

### Enable in SSH

```bash
sudo nano /etc/ssh/sshd_config
```

```bash
ChallengeResponseAuthentication yes
AuthenticationMethods publickey,keyboard-interactive
```

Restart SSH:
```bash
sudo systemctl restart sshd
```

---

## Port Knocking (Advanced)

Hide SSH port until knocked.

### Install knockd

```bash
sudo apt install knockd -y
```

### Configure

```bash
sudo nano /etc/knockd.conf
```

```ini
[options]
    UseSyslog

[openSSH]
    sequence    = 7000,8000,9000
    seq_timeout = 5
    command     = /sbin/iptables -A INPUT -s %IP% -p tcp --dport 22 -j ACCEPT
    tcpflags    = syn

[closeSSH]
    sequence    = 9000,8000,7000
    seq_timeout = 5
    command     = /sbin/iptables -D INPUT -s %IP% -p tcp --dport 22 -j ACCEPT
    tcpflags    = syn
```

### Enable knockd

```bash
sudo nano /etc/default/knockd
```

```bash
START_KNOCKD=1
KNOCKD_OPTS="-i eth0"
```

```bash
sudo systemctl enable knockd
sudo systemctl start knockd
```

### Client Usage

```bash
# Open SSH port
knock server-ip 7000 8000 9000
ssh user@server-ip

# Close SSH port
knock server-ip 9000 8000 7000
```

---

## Security Checklist

- [ ] Use Ed25519 or RSA 4096-bit keys
- [ ] Disable password authentication
- [ ] Disable root login
- [ ] Use non-standard SSH port
- [ ] Set up Fail2Ban
- [ ] Configure SSH timeouts
- [ ] Use SSH config file for convenience
- [ ] Enable key passphrase
- [ ] Consider 2FA for critical servers
- [ ] Regular key rotation
- [ ] Monitor /var/log/auth.log

---

## Troubleshooting

### Permission Denied (publickey)

```bash
# Check permissions
chmod 700 ~/.ssh
chmod 600 ~/.ssh/authorized_keys
chmod 600 ~/.ssh/id_ed25519
chmod 644 ~/.ssh/id_ed25519.pub

# Check ownership
ls -la ~/.ssh

# Verbose connection
ssh -vvv user@server
```

### Connection Refused

```bash
# Check if SSH is running
sudo systemctl status sshd

# Check port
sudo ss -tlnp | grep ssh

# Check firewall
sudo ufw status
```

### Key Not Being Used

```bash
# Specify key explicitly
ssh -i ~/.ssh/id_ed25519 user@server

# Check loaded keys
ssh-add -l
```

---

## Next Steps

1. **[Firewall Setup](./02-firewall-configuration.md)** - UFW/iptables configuration
2. **[Web Server](./nginx-complete-guide.md)** - Nginx setup
3. **[SSL/HTTPS](./certbot-installation.md)** - Certbot installation

---

✅ Your SSH is now secured!
