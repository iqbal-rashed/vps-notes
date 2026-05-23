# Get a Free SSL Certificate (HTTPS)

SSL makes your site use `https://` instead of `http://`. It's free with Let's Encrypt and takes 2 minutes to set up.

**Before you start:**
- Your domain must point to your server (DNS A record set up)
- Nginx must be installed and running
- Ports 80 and 443 must be open in your firewall

---

## Step 1 — Install Certbot

```bash
sudo apt install certbot python3-certbot-nginx -y
```

---

## Step 2 — Get Your Certificate

Replace `example.com` with your actual domain:

```bash
sudo certbot --nginx -d example.com -d www.example.com
```

Certbot will ask you a few questions:
1. Your email address (for renewal reminders)
2. Agree to terms — type `A`
3. Share email with EFF — type `N` (your choice)
4. Redirect HTTP to HTTPS — type `2` (recommended)

That's it! Your site now has HTTPS.

---

## Auto-Renewal

Certbot automatically renews certificates before they expire. Test it works:

```bash
sudo certbot renew --dry-run
```

If no errors, you're good. Certbot will renew automatically.

---

## Useful Commands

```bash
# See your certificates
sudo certbot certificates

# Manually renew all certificates
sudo certbot renew

# Add more domains to an existing certificate
sudo certbot --nginx --expand -d example.com -d www.example.com -d api.example.com

# Remove a certificate
sudo certbot delete --cert-name example.com
```

---

## Troubleshooting

**"DNS problem: NXDOMAIN" error** — Your domain doesn't point to this server yet.
Check your DNS:
```bash
dig example.com +short
# Should show your server IP
```

**"Port 80 not accessible"** — Check your firewall:
```bash
sudo ufw status
sudo ufw allow 80
sudo ufw allow 443
```

**Certificate not renewing** — Check the renewal config:
```bash
sudo certbot renew --dry-run
sudo cat /var/log/letsencrypt/letsencrypt.log
```

---

## Next Steps

1. [Deploy Node.js App](./deploy-nodejs-app.md)
2. [Deploy Next.js App](./deploy-nextjs-app.md)
3. [Server Security](./server-security-hardening.md)
