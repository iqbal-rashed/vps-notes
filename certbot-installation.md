# Install Certbot on VPS with Apache or Nginx

This guide explains how to install and configure **Certbot** on a VPS to
enable HTTPS with **Apache** or **Nginx**.

------------------------------------------------------------------------

## 1. Update Your Server

Before installing, make sure your system is up to date:

``` bash
sudo apt update && sudo apt upgrade -y
```

------------------------------------------------------------------------

## 2. Install Certbot

Certbot is available via your system's package manager.

``` bash
sudo apt install certbot python3-certbot-apache -y   # For Apache
sudo apt install certbot python3-certbot-nginx -y    # For Nginx
```

------------------------------------------------------------------------

## 3. Obtain SSL Certificate

### For Apache:

``` bash
sudo certbot --apache
```

### For Nginx:

``` bash
sudo certbot --nginx
```

You will be prompted to: - Enter your email address (for renewal
notices). - Agree to Let's Encrypt terms of service. - Choose whether to
redirect all HTTP traffic to HTTPS.

------------------------------------------------------------------------

## 4. Auto-Renewal

Certbot sets up automatic renewal. Test it with:

``` bash
sudo certbot renew --dry-run
```

------------------------------------------------------------------------

## 5. Verify Installation

-   Open your domain in a browser with `https://yourdomain.com`
-   Use SSL Labs to check: <https://www.ssllabs.com/ssltest/>

------------------------------------------------------------------------

## Troubleshooting

-   Ensure your domain DNS points to the VPS IP.
-   Open firewall ports **80 (HTTP)** and **443 (HTTPS)**:

``` bash
sudo ufw allow 80
sudo ufw allow 443
sudo ufw allow 22 // ssh
sudo ufw enable
sudo ufw reload

```

------------------------------------------------------------------------

✅ Now your VPS is secured with a free SSL certificate from **Let's
Encrypt**!
