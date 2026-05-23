# Nginx Setup Guide

Nginx is a web server. It receives requests from visitors and either:
- Serves your website files directly (for static sites)
- Forwards requests to your app (Node.js, Next.js, etc.)

---

## Install Nginx

```bash
sudo apt update
sudo apt install nginx -y

# Start it and enable auto-start on reboot
sudo systemctl enable nginx
sudo systemctl start nginx

# Check it's running
sudo systemctl status nginx
```

Visit `http://YOUR_SERVER_IP` in your browser — you should see the Nginx welcome page.

---

## Basic Commands

```bash
# Start / Stop / Restart
sudo systemctl start nginx
sudo systemctl stop nginx
sudo systemctl restart nginx

# Reload config (no downtime)
sudo systemctl reload nginx

# Test your config for errors before reloading
sudo nginx -t

# View error log
sudo tail -f /var/log/nginx/error.log
```

---

## How Nginx Config Files Work

```
/etc/nginx/
├── nginx.conf              ← Main config (don't change much)
├── sites-available/        ← All your site configs go here
└── sites-enabled/          ← Links to the ones that are active
```

**Workflow:**
1. Create a config file in `sites-available/`
2. Link it to `sites-enabled/` to activate it
3. Test the config and reload Nginx

---

## Setup for a Static Website

For HTML/CSS/JS files (React build, plain HTML, etc.):

```bash
# Create folder for your site
sudo mkdir -p /var/www/example.com
sudo chown -R $USER:$USER /var/www/example.com

# Create the Nginx config
sudo nano /etc/nginx/sites-available/example.com
```

Paste this (replace `example.com` with your domain):

```nginx
server {
    listen 80;
    server_name example.com www.example.com;
    root /var/www/example.com;
    index index.html;

    location / {
        try_files $uri $uri/ /index.html;
    }

    # Cache images, CSS, JS files for 30 days
    location ~* \.(jpg|jpeg|png|gif|ico|css|js|woff|woff2)$ {
        expires 30d;
    }

    # Block hidden files (like .htaccess, .git)
    location ~ /\. {
        deny all;
    }
}
```

Enable it:
```bash
sudo ln -s /etc/nginx/sites-available/example.com /etc/nginx/sites-enabled/
sudo nginx -t
sudo systemctl reload nginx
```

---

## Setup for a Node.js / Express App

Your Node.js app runs on port 3000. Nginx forwards traffic from port 80 to it.

```bash
sudo nano /etc/nginx/sites-available/myapp
```

```nginx
server {
    listen 80;
    server_name example.com www.example.com;

    location / {
        proxy_pass http://localhost:3000;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

Enable it:
```bash
sudo ln -s /etc/nginx/sites-available/myapp /etc/nginx/sites-enabled/
sudo nginx -t
sudo systemctl reload nginx
```

---

## Setup for a Next.js App (Server-Side)

Next.js runs as a server on port 3000. Nginx forwards traffic to it.

```bash
sudo nano /etc/nginx/sites-available/nextjs-app
```

```nginx
server {
    listen 80;
    server_name example.com www.example.com;

    # Static Next.js files (CSS, JS, images) — cached for 1 year
    location /_next/static/ {
        proxy_pass http://localhost:3000;
        expires 365d;
        add_header Cache-Control "public, immutable";
        access_log off;
    }

    # API routes — never cached
    location /api/ {
        proxy_pass http://localhost:3000;
        proxy_http_version 1.1;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }

    # Everything else — forward to Next.js server
    location / {
        proxy_pass http://localhost:3000;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

Enable it:
```bash
sudo ln -s /etc/nginx/sites-available/nextjs-app /etc/nginx/sites-enabled/
sudo nginx -t
sudo systemctl reload nginx
```

---

## Add Security Headers

Add these to any `server {}` block to improve security:

```nginx
# Stop browsers from guessing file types
add_header X-Content-Type-Options "nosniff" always;

# Don't allow your site to be put in an iframe on other sites
add_header X-Frame-Options "SAMEORIGIN" always;

# Enable browser XSS protection
add_header X-XSS-Protection "1; mode=block" always;

# Control how much info is shared with other sites
add_header Referrer-Policy "strict-origin-when-cross-origin" always;

# Hide Nginx version number
server_tokens off;
```

Put `server_tokens off;` in the `http {}` block of `/etc/nginx/nginx.conf`.

---

## Enable Gzip Compression

Makes your site load faster by compressing files:

Add to the `http {}` block in `/etc/nginx/nginx.conf`:

```nginx
gzip on;
gzip_types text/plain text/css application/json application/javascript text/xml image/svg+xml;
gzip_comp_level 5;
```

---

## Troubleshooting

**Test your config:**
```bash
sudo nginx -t
```

**Check error log:**
```bash
sudo tail -50 /var/log/nginx/error.log
```

**502 Bad Gateway** — Your app isn't running. Check:
```bash
pm2 status
sudo systemctl status your-app
```

**Permission denied** — Fix file ownership:
```bash
sudo chown -R www-data:www-data /var/www/example.com
```

---

## Next Steps

1. [Get SSL Certificate](./certbot-installation.md) — Add HTTPS to your site
2. [Deploy Node.js](./deploy-nodejs-app.md) — Deploy a Node.js app
3. [Deploy Next.js](./deploy-nextjs-app.md) — Deploy a Next.js app
