# Server Security Hardening

Complete security checklist for production servers.

---

## 1. User Security

```bash
# Disable root login
sudo passwd -l root

# Use strong passwords
# Or better: use SSH keys only

# Remove unused users
sudo userdel username
```

---

## 2. SSH Hardening

Edit `/etc/ssh/sshd_config`:

```bash
Port 2222                    # Change default port
PermitRootLogin no
PasswordAuthentication no
PubkeyAuthentication yes
MaxAuthTries 3
ClientAliveInterval 300
AllowUsers deployer
```

```bash
sudo systemctl restart sshd
```

---

## 3. Firewall

```bash
# UFW setup
sudo ufw default deny incoming
sudo ufw default allow outgoing
sudo ufw allow 2222/tcp    # SSH
sudo ufw allow 80/tcp      # HTTP
sudo ufw allow 443/tcp     # HTTPS
sudo ufw enable
```

---

## 4. Fail2Ban

```bash
sudo apt install fail2ban -y
sudo nano /etc/fail2ban/jail.local
```

```ini
[sshd]
enabled = true
port = 2222
maxretry = 3
bantime = 24h
```

```bash
sudo systemctl restart fail2ban
```

---

## 5. Automatic Updates

```bash
sudo apt install unattended-upgrades -y
sudo dpkg-reconfigure -plow unattended-upgrades
```

---

## 6. Security Headers (Nginx)

```nginx
add_header X-Frame-Options "SAMEORIGIN" always;
add_header X-Content-Type-Options "nosniff" always;
add_header X-XSS-Protection "1; mode=block" always;
add_header Referrer-Policy "strict-origin-when-cross-origin" always;
add_header Content-Security-Policy "default-src 'self'" always;
add_header Strict-Transport-Security "max-age=31536000" always;
```

---

## 7. File Permissions

```bash
# Web files
sudo chown -R www-data:www-data /var/www
sudo chmod -R 755 /var/www
sudo chmod 600 /var/www/app/.env

# SSH keys
chmod 700 ~/.ssh
chmod 600 ~/.ssh/authorized_keys
```

---

## 8. Disable Unused Services

```bash
# List services
sudo systemctl list-unit-files --type=service

# Disable unused
sudo systemctl disable cups
sudo systemctl disable bluetooth
```

---

## 9. Monitoring

```bash
# Check auth logs
sudo tail -f /var/log/auth.log

# Failed logins
sudo grep "Failed password" /var/log/auth.log

# Install rootkit checker
sudo apt install rkhunter -y
sudo rkhunter --check
```

---

## Security Checklist

- [ ] Non-root user with sudo
- [ ] SSH key authentication only
- [ ] Changed SSH port
- [ ] Firewall enabled (UFW)
- [ ] Fail2Ban installed
- [ ] Automatic updates enabled
- [ ] SSL/HTTPS configured
- [ ] Security headers set
- [ ] File permissions correct
- [ ] Unused services disabled
- [ ] Regular backups scheduled
- [ ] Logs monitored

---

✅ Server hardened!
