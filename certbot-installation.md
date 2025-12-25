# Install Certbot for SSL/HTTPS

Complete guide for installing and configuring **Certbot** to enable free SSL certificates from **Let's Encrypt** on your VPS.

---

## Prerequisites

- A domain name pointing to your server (DNS A record)
- Web server installed (Nginx or Apache)
- Ports 80 and 443 open in firewall

---

## 1. Update Your Server

Before installing, make sure your system is up to date:

```bash
sudo apt update && sudo apt upgrade -y
```

---

## 2. Install Certbot

### For Nginx

```bash
sudo apt install certbot python3-certbot-nginx -y
```

### For Apache

```bash
sudo apt install certbot python3-certbot-apache -y
```

### Standalone (No Web Server)

```bash
sudo apt install certbot -y
```

---

## 3. Obtain SSL Certificate

### Nginx (Automatic Configuration)

```bash
sudo certbot --nginx -d example.com -d www.example.com
```

### Apache (Automatic Configuration)

```bash
sudo certbot --apache -d example.com -d www.example.com
```

### Standalone Mode

Stop your web server first, then:

```bash
sudo certbot certonly --standalone -d example.com -d www.example.com
```

### Webroot Mode (No Downtime)

```bash
sudo certbot certonly --webroot -w /var/www/example.com -d example.com -d www.example.com
```

### DNS Challenge (Wildcard Certificates)

```bash
sudo certbot certonly --manual --preferred-challenges dns -d "*.example.com" -d example.com
```

You'll be prompted to add a TXT record to your DNS.

---

## 4. Certificate Prompts

During the process, you'll be asked:

1. **Email address** - For renewal notices and security alerts
2. **Terms of Service** - Agree to Let's Encrypt ToS
3. **Newsletter** - Optional, share email with EFF
4. **HTTP to HTTPS redirect** - Choose whether to redirect all traffic

---

## 5. Certificate Files

After successful issuance, certificates are stored in:

```
/etc/letsencrypt/live/example.com/
├── cert.pem       # Domain certificate
├── chain.pem      # Intermediate certificate
├── fullchain.pem  # cert.pem + chain.pem (use this)
└── privkey.pem    # Private key
```

---

## 6. Auto-Renewal

Certbot sets up automatic renewal via systemd timer or cron.

### Test Renewal

```bash
sudo certbot renew --dry-run
```

### Check Timer Status

```bash
sudo systemctl status certbot.timer
sudo systemctl list-timers | grep certbot
```

### Manual Renewal

```bash
sudo certbot renew
```

### Force Renewal

```bash
sudo certbot renew --force-renewal
```

---

## 7. Verify SSL

### Browser Check

Visit `https://yourdomain.com` and click the padlock icon.

### SSL Labs Test

Visit: https://www.ssllabs.com/ssltest/

### Command Line Check

```bash
# Check certificate dates
sudo certbot certificates

# OpenSSL check
openssl s_client -connect example.com:443 -servername example.com

# Check expiry
echo | openssl s_client -connect example.com:443 2>/dev/null | openssl x509 -noout -dates
```

---

## 8. Manual Nginx SSL Configuration

If Certbot doesn't configure automatically:

```nginx
server {
    listen 80;
    server_name example.com www.example.com;
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    listen [::]:443 ssl http2;
    server_name example.com www.example.com;

    # SSL Certificates
    ssl_certificate /etc/letsencrypt/live/example.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/example.com/privkey.pem;

    # SSL Configuration
    ssl_session_timeout 1d;
    ssl_session_cache shared:SSL:50m;
    ssl_session_tickets off;

    # Modern protocols only
    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_ciphers ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-CHACHA20-POLY1305:ECDHE-RSA-CHACHA20-POLY1305;
    ssl_prefer_server_ciphers off;

    # HSTS (2 years)
    add_header Strict-Transport-Security "max-age=63072000" always;

    # OCSP Stapling
    ssl_stapling on;
    ssl_stapling_verify on;
    ssl_trusted_certificate /etc/letsencrypt/live/example.com/chain.pem;
    resolver 8.8.8.8 8.8.4.4 valid=300s;

    # Your site configuration
    root /var/www/example.com/html;
    index index.html;

    location / {
        try_files $uri $uri/ =404;
    }
}
```

Reload Nginx:

```bash
sudo nginx -t
sudo systemctl reload nginx
```

---

## 9. Multiple Domains

### Single Certificate for Multiple Domains

```bash
sudo certbot --nginx -d example.com -d www.example.com -d api.example.com -d app.example.com
```

### Separate Certificates

```bash
sudo certbot --nginx -d example.com -d www.example.com
sudo certbot --nginx -d api.example.com
```

### Expand Existing Certificate

```bash
sudo certbot --nginx --expand -d example.com -d www.example.com -d new.example.com
```

---

## 10. Wildcard Certificates

Wildcard certificates cover all subdomains:

```bash
sudo certbot certonly --manual --preferred-challenges dns -d "*.example.com" -d example.com
```

### With DNS Plugin (Automated)

For Cloudflare:
```bash
sudo apt install python3-certbot-dns-cloudflare -y
```

Create credentials file:
```bash
sudo nano /etc/letsencrypt/cloudflare.ini
```

```ini
dns_cloudflare_email = your-email@example.com
dns_cloudflare_api_key = your-global-api-key
```

```bash
sudo chmod 600 /etc/letsencrypt/cloudflare.ini
sudo certbot certonly --dns-cloudflare --dns-cloudflare-credentials /etc/letsencrypt/cloudflare.ini -d "*.example.com" -d example.com
```

---

## 11. Renewal Hooks

Run commands before/after renewal:

```bash
sudo nano /etc/letsencrypt/renewal-hooks/deploy/reload-nginx.sh
```

```bash
#!/bin/bash
systemctl reload nginx
```

```bash
sudo chmod +x /etc/letsencrypt/renewal-hooks/deploy/reload-nginx.sh
```

---

## 12. Revoke and Delete Certificates

### Revoke

```bash
sudo certbot revoke --cert-path /etc/letsencrypt/live/example.com/cert.pem
```

### Delete

```bash
sudo certbot delete --cert-name example.com
```

---

## Troubleshooting

### DNS Not Pointing to Server

```bash
# Check DNS
dig example.com +short
nslookup example.com

# Should return your server IP
```

### Port 80 Not Accessible

```bash
# Check firewall
sudo ufw status
sudo ufw allow 80
sudo ufw allow 443

# Check if something else is using port 80
sudo ss -tlnp | grep :80
```

### Rate Limits

Let's Encrypt has rate limits:
- 50 certificates per domain per week
- 5 duplicate certificates per week
- 5 failed validations per hour

Use staging for testing:
```bash
sudo certbot --nginx --staging -d example.com
```

### Certificate Not Renewing

```bash
# Check renewal config
sudo cat /etc/letsencrypt/renewal/example.com.conf

# Manual test
sudo certbot renew --dry-run

# Check logs
sudo cat /var/log/letsencrypt/letsencrypt.log
```

---

## Certbot Commands Reference

| Command | Description |
|---------|-------------|
| `certbot --nginx` | Obtain and install for Nginx |
| `certbot --apache` | Obtain and install for Apache |
| `certbot certonly` | Obtain only (manual install) |
| `certbot renew` | Renew all certificates |
| `certbot renew --dry-run` | Test renewal |
| `certbot certificates` | List certificates |
| `certbot delete` | Delete certificate |
| `certbot revoke` | Revoke certificate |

---

## Security Best Practices

1. **Enable HSTS** - Force HTTPS for browsers
2. **Use TLS 1.2+** - Disable older protocols
3. **Regular renewals** - Certificates expire in 90 days
4. **Monitor expiry** - Set up alerting
5. **Test regularly** - Use SSL Labs

---

## Next Steps

1. **[Nginx Configuration](./nginx-complete-guide.md)** - Advanced Nginx setup
2. **[Security Hardening](./server-security-hardening.md)** - Full server security
3. **[Monitoring](./server-monitoring-guide.md)** - Monitor your server

---

✅ Your site is now secured with free SSL from **Let's Encrypt**!
