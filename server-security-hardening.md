# Server Security Hardening

Extra security steps to protect your server. Do these after the basic setup.

Most of these are simple one-time commands. Each one closes a potential hole attackers can use.

---

## 1. Lock the Root Account

Root is the most powerful account. Lock it so nobody can log in as root directly.

```bash
sudo passwd -l root
```

---

## 2. Keep the System Updated

Old software has known security holes. Update regularly.

```bash
sudo apt update && sudo apt upgrade -y
```

Set up automatic security updates (if not done already):
```bash
sudo apt install unattended-upgrades -y
sudo dpkg-reconfigure -plow unattended-upgrades
```

---

## 3. SSH Hardening

Make sure your `/etc/ssh/sshd_config` has these settings:

```bash
sudo nano /etc/ssh/sshd_config
```

```
Port 2222
PermitRootLogin no
PasswordAuthentication no
PubkeyAuthentication yes
MaxAuthTries 3
AllowUsers deployer
ClientAliveInterval 300
ClientAliveCountMax 2
LoginGraceTime 30
```

Apply changes:
```bash
sudo systemctl restart sshd
```

---

## 4. Firewall (UFW)

Only allow ports you actually need:

```bash
sudo ufw default deny incoming
sudo ufw default allow outgoing
sudo ufw allow 2222/tcp   # SSH
sudo ufw allow 80/tcp     # HTTP
sudo ufw allow 443/tcp    # HTTPS
sudo ufw enable
```

See [Firewall Guide](./02-firewall-configuration.md) for full details.

---

## 5. Fail2Ban (Block Brute Force)

Fail2Ban automatically bans IPs that try to break in.

```bash
sudo apt install fail2ban -y
sudo systemctl enable fail2ban
sudo systemctl start fail2ban
```

See [SSH Security Guide](./03-ssh-keys-and-security.md) for full Fail2Ban config.

---

## 6. Secure Shared Memory

Shared memory can be exploited by attackers. Mount it as read-only:

```bash
sudo nano /etc/fstab
```

Add this line at the bottom:
```
tmpfs /run/shm tmpfs defaults,noexec,nosuid 0 0
```

Apply:
```bash
sudo mount -a
```

---

## 7. Protect Important System Files

Make key config files read-only:

```bash
# Lock the hosts file
sudo chattr +i /etc/hosts

# Lock the hostname file
sudo chattr +i /etc/hostname
```

To unlock later if you need to edit:
```bash
sudo chattr -i /etc/hosts
```

---

## 8. Limit Who Can Use sudo

Check who has sudo access:
```bash
sudo cat /etc/sudoers
grep -r 'sudo' /etc/group
```

Remove any users who don't need it:
```bash
sudo deluser username sudo
```

---

## 9. Check for Open Ports

See what's listening on your server. Any unexpected open ports are a red flag.

```bash
sudo ss -tlnp
```

You should only see ports you expect (22/2222, 80, 443, 3000 for your app, etc.).

---

## 10. Disable Unused Services

Check what services are running:
```bash
sudo systemctl list-units --type=service --state=running
```

Disable services you don't need (example):
```bash
sudo systemctl disable bluetooth
sudo systemctl stop bluetooth
```

---

## 11. Set Secure File Permissions

Your app's `.env` file should never be readable by other users:
```bash
chmod 600 /var/www/myapp/.env
```

Web files should be readable by Nginx but not writable:
```bash
sudo chown -R www-data:www-data /var/www/example.com
sudo find /var/www/example.com -type f -exec chmod 644 {} \;
sudo find /var/www/example.com -type d -exec chmod 755 {} \;
```

---

## 12. Add Security Headers to Nginx

Put these in your Nginx server block:

```nginx
add_header X-Content-Type-Options "nosniff" always;
add_header X-Frame-Options "SAMEORIGIN" always;
add_header X-XSS-Protection "1; mode=block" always;
add_header Referrer-Policy "strict-origin-when-cross-origin" always;
server_tokens off;
```

---

## 13. Check Login History

See who has logged into your server:
```bash
# Recent logins
last

# Failed login attempts
sudo grep "Failed password" /var/log/auth.log | tail -20

# Currently logged in users
who
```

---

## Quick Security Checklist

- [ ] Root account locked
- [ ] System updated + auto-updates on
- [ ] SSH: root login disabled, password login disabled, port changed
- [ ] Fail2Ban installed and running
- [ ] Firewall on — only needed ports open
- [ ] Databases NOT exposed to the internet
- [ ] `.env` files have permission `600`
- [ ] Security headers added to Nginx
- [ ] Check open ports — nothing unexpected

---

## Next Steps

1. [Monitoring Guide](./server-monitoring-guide.md) — Get alerts when something goes wrong
2. [Backup Strategies](./backup-strategies.md) — Back up your data regularly
