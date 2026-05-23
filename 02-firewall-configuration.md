# Firewall Setup

A firewall blocks all connections except the ones you allow. This keeps hackers out.

We use **UFW** (Uncomplicated Firewall) — it's the easiest option on Ubuntu.

---

## Step 1 — Install UFW

```bash
sudo apt install ufw -y
```

---

## Step 2 — Set Default Rules

Block everything coming in. Allow everything going out.

```bash
sudo ufw default deny incoming
sudo ufw default allow outgoing
```

---

## Step 3 — Allow the Ports You Need

> ⚠️ **Allow SSH first or you'll lock yourself out!**

```bash
# If you changed SSH to port 2222 (recommended):
sudo ufw allow 2222/tcp

# If you kept default port 22:
sudo ufw allow 22/tcp

# Web server
sudo ufw allow 80/tcp    # HTTP
sudo ufw allow 443/tcp   # HTTPS
```

---

## Step 4 — Turn On the Firewall

```bash
sudo ufw enable
```

Type `y` and press Enter when asked.

---

## Step 5 — Check What's Open

```bash
sudo ufw status
```

You should see something like:
```
Status: active

To                         Action      From
--                         ------      ----
2222/tcp                   ALLOW       Anywhere
80/tcp                     ALLOW       Anywhere
443/tcp                    ALLOW       Anywhere
```

---

## Useful Commands

```bash
# Check status
sudo ufw status

# Allow a port
sudo ufw allow 3000/tcp

# Block a port
sudo ufw deny 3306/tcp

# Remove a rule
sudo ufw delete allow 3000/tcp

# Disable firewall temporarily
sudo ufw disable

# Reset all rules (start over)
sudo ufw reset
```

---

## Protect Databases (Important!)

Never open database ports to the public internet. They should only be accessible from localhost (your own server).

```bash
# Block external access to databases
sudo ufw deny 3306/tcp   # MySQL
sudo ufw deny 5432/tcp   # PostgreSQL
sudo ufw deny 27017/tcp  # MongoDB
sudo ufw deny 6379/tcp   # Redis
```

These apps already listen on localhost only by default — but blocking them at the firewall adds extra protection.

---

## Limit SSH Brute Force Attacks

UFW can automatically block IPs that try to connect too many times:

```bash
# Block IPs that fail login more than 6 times in 30 seconds
sudo ufw limit 2222/tcp
```

---

## Common Ports Reference

| Port | What It's For |
|------|--------------|
| 22 / 2222 | SSH (server management) |
| 80 | Website (HTTP) |
| 443 | Website (HTTPS) |
| 3000 | Node.js app |
| 3306 | MySQL |
| 5432 | PostgreSQL |
| 27017 | MongoDB |
| 6379 | Redis |

---

## Locked Out of Your Server?

If you accidentally blocked yourself:

- **VPS:** Log in via your provider's web console (Hetzner, DigitalOcean, etc.)
- **AWS EC2:** Edit the Security Group in your AWS Console, then re-add port 22/2222

---

## Next Steps

1. [SSH Security](./03-ssh-keys-and-security.md) — Stop brute force attacks with Fail2Ban
2. [Nginx Setup](./nginx-complete-guide.md) — Install your web server
3. [SSL/HTTPS](./certbot-installation.md) — Get a free SSL certificate
