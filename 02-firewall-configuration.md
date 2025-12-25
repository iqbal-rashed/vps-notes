# Firewall Configuration Guide

Complete guide for configuring UFW, iptables, and AWS Security Groups.

---

## UFW (Uncomplicated Firewall) - Recommended for Ubuntu

UFW is the easiest way to manage firewall rules on Ubuntu/Debian.

### Install UFW

```bash
sudo apt install ufw -y
```

### Basic Commands

```bash
# Check status
sudo ufw status
sudo ufw status verbose

# Enable/Disable
sudo ufw enable
sudo ufw disable

# Reset to defaults
sudo ufw reset
```

### Essential Rules

```bash
# Allow SSH (IMPORTANT: Do this before enabling UFW!)
sudo ufw allow 22/tcp
# Or if using custom port:
sudo ufw allow 2222/tcp

# Allow HTTP and HTTPS
sudo ufw allow 80/tcp
sudo ufw allow 443/tcp

# Allow specific port
sudo ufw allow 3000/tcp

# Allow port range
sudo ufw allow 8000:8100/tcp

# Allow from specific IP
sudo ufw allow from 192.168.1.100

# Allow from specific IP to specific port
sudo ufw allow from 192.168.1.100 to any port 22

# Allow from subnet
sudo ufw allow from 192.168.1.0/24

# Deny specific port
sudo ufw deny 3306/tcp

# Delete a rule
sudo ufw delete allow 80/tcp
# Or by number:
sudo ufw status numbered
sudo ufw delete 2
```

### Common Application Profiles

```bash
# List available profiles
sudo ufw app list

# Allow Nginx
sudo ufw allow 'Nginx Full'    # HTTP + HTTPS
sudo ufw allow 'Nginx HTTP'    # HTTP only
sudo ufw allow 'Nginx HTTPS'   # HTTPS only

# Allow OpenSSH
sudo ufw allow OpenSSH

# Allow Apache
sudo ufw allow 'Apache Full'
```

### Recommended Configuration

```bash
# Default policies
sudo ufw default deny incoming
sudo ufw default allow outgoing

# Essential services
sudo ufw allow ssh
sudo ufw allow 'Nginx Full'

# Enable firewall
sudo ufw enable
```

### Rate Limiting (Prevent Brute Force)

```bash
# Limit SSH connections (max 6 per 30 seconds)
sudo ufw limit ssh

# Or for custom port
sudo ufw limit 2222/tcp
```

---

## iptables (Advanced)

For more advanced firewall control.

### Basic Commands

```bash
# List rules
sudo iptables -L -v -n
sudo iptables -L --line-numbers

# Save rules
sudo iptables-save > /etc/iptables.rules

# Restore rules
sudo iptables-restore < /etc/iptables.rules
```

### Common Rules

```bash
# Allow established connections
sudo iptables -A INPUT -m state --state ESTABLISHED,RELATED -j ACCEPT

# Allow loopback
sudo iptables -A INPUT -i lo -j ACCEPT

# Allow SSH
sudo iptables -A INPUT -p tcp --dport 22 -j ACCEPT

# Allow HTTP/HTTPS
sudo iptables -A INPUT -p tcp --dport 80 -j ACCEPT
sudo iptables -A INPUT -p tcp --dport 443 -j ACCEPT

# Drop all other incoming
sudo iptables -A INPUT -j DROP

# Allow specific port
sudo iptables -A INPUT -p tcp --dport 3000 -j ACCEPT

# Allow from specific IP
sudo iptables -A INPUT -s 192.168.1.100 -j ACCEPT

# Block IP
sudo iptables -A INPUT -s 192.168.1.100 -j DROP

# Delete rule by line number
sudo iptables -D INPUT 3
```

### Make iptables Persistent

```bash
# Install persistence package
sudo apt install iptables-persistent -y

# Save current rules
sudo netfilter-persistent save

# Rules are stored in:
# /etc/iptables/rules.v4
# /etc/iptables/rules.v6
```

---

## AWS Security Groups

Security Groups act as virtual firewalls for EC2 instances.

### Key Concepts

- **Stateful**: Return traffic is automatically allowed
- **Default Deny**: All inbound traffic is denied by default
- **Allow Only**: You can only create allow rules, not deny rules

### Common Inbound Rules

| Type | Protocol | Port | Source | Description |
|------|----------|------|--------|-------------|
| SSH | TCP | 22 | My IP | SSH access |
| HTTP | TCP | 80 | 0.0.0.0/0 | Web traffic |
| HTTPS | TCP | 443 | 0.0.0.0/0 | Secure web traffic |
| Custom TCP | TCP | 3000 | 0.0.0.0/0 | Node.js app |
| MySQL/Aurora | TCP | 3306 | sg-xxxxx | Database from app SG |
| PostgreSQL | TCP | 5432 | sg-xxxxx | Database from app SG |
| MongoDB | TCP | 27017 | sg-xxxxx | Database from app SG |
| Redis | TCP | 6379 | sg-xxxxx | Cache from app SG |

### Best Practices for AWS

```
# Instead of:
Source: 0.0.0.0/0

# Use:
Source: Your-IP/32 (for SSH)
Source: Web-Server-SG (for database)
```

### AWS CLI Commands

```bash
# List security groups
aws ec2 describe-security-groups

# Create security group
aws ec2 create-security-group \
    --group-name web-server-sg \
    --description "Web server security group" \
    --vpc-id vpc-xxxxx

# Add inbound rule
aws ec2 authorize-security-group-ingress \
    --group-id sg-xxxxx \
    --protocol tcp \
    --port 443 \
    --cidr 0.0.0.0/0

# Remove inbound rule
aws ec2 revoke-security-group-ingress \
    --group-id sg-xxxxx \
    --protocol tcp \
    --port 22 \
    --cidr 0.0.0.0/0
```

---

## Firewall Rules by Service

### Web Applications

```bash
sudo ufw allow 80/tcp     # HTTP
sudo ufw allow 443/tcp    # HTTPS
sudo ufw allow 3000/tcp   # Node.js (dev)
sudo ufw allow 8080/tcp   # Alternative HTTP
```

### Databases (Allow only from localhost or app servers)

```bash
# MySQL - only from specific IP
sudo ufw allow from 10.0.0.5 to any port 3306

# PostgreSQL - only from specific IP
sudo ufw allow from 10.0.0.5 to any port 5432

# MongoDB - only from localhost
sudo ufw allow from 127.0.0.1 to any port 27017

# Redis - only from localhost
sudo ufw allow from 127.0.0.1 to any port 6379
```

### Email Servers

```bash
sudo ufw allow 25/tcp     # SMTP
sudo ufw allow 587/tcp    # SMTP (TLS)
sudo ufw allow 993/tcp    # IMAP (TLS)
sudo ufw allow 995/tcp    # POP3 (TLS)
```

### FTP (Not Recommended - Use SFTP)

```bash
sudo ufw allow 21/tcp
sudo ufw allow 20/tcp
```

---

## Troubleshooting

### Check Open Ports

```bash
# Check listening ports
sudo ss -tlnp
sudo netstat -tlnp

# Check if port is accessible
nc -zv your-server-ip 80
telnet your-server-ip 80

# Check from outside
nmap -p 22,80,443 your-server-ip
```

### Debug UFW

```bash
# Check UFW logs
sudo tail -f /var/log/ufw.log

# Enable logging
sudo ufw logging on
sudo ufw logging medium
```

### Locked Out of Server?

If you've locked yourself out:

1. **AWS EC2**: Modify Security Group via AWS Console
2. **VPS**: Use provider's console/VNC access
3. **Rescue Mode**: Boot into rescue mode via provider dashboard

---

## Quick Reference

### UFW Status Check

```bash
sudo ufw status verbose
```

### Common Ports

| Port | Service |
|------|---------|
| 22 | SSH |
| 80 | HTTP |
| 443 | HTTPS |
| 3000 | Node.js |
| 3306 | MySQL |
| 5432 | PostgreSQL |
| 27017 | MongoDB |
| 6379 | Redis |

---

## Next Steps

1. **[SSH Security](./03-ssh-keys-and-security.md)** - Fail2ban and advanced SSH
2. **[SSL Setup](./certbot-installation.md)** - HTTPS with Let's Encrypt
3. **[Web Server](./nginx-complete-guide.md)** - Nginx configuration

---

✅ Your firewall is now configured!
