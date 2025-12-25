# Nginx Complete Guide

Complete guide for Nginx installation, configuration, virtual hosts, reverse proxy, and optimization.

---

## Installation

### Ubuntu/Debian

```bash
sudo apt update
sudo apt install nginx -y
```

### CentOS/RHEL

```bash
sudo yum install epel-release -y
sudo yum install nginx -y
```

### Verify Installation

```bash
nginx -v
sudo systemctl status nginx
```

---

## Basic Commands

```bash
# Start/Stop/Restart
sudo systemctl start nginx
sudo systemctl stop nginx
sudo systemctl restart nginx

# Reload (no downtime)
sudo systemctl reload nginx

# Enable at boot
sudo systemctl enable nginx

# Test configuration
sudo nginx -t

# View error logs
sudo tail -f /var/log/nginx/error.log

# View access logs
sudo tail -f /var/log/nginx/access.log
```

---

## Directory Structure

```
/etc/nginx/
├── nginx.conf              # Main configuration
├── sites-available/        # Available site configs
├── sites-enabled/          # Enabled site configs (symlinks)
├── conf.d/                 # Additional configs
├── snippets/               # Reusable config snippets
└── mime.types              # MIME type definitions

/var/www/                   # Web root (default)
/var/log/nginx/             # Log files
```

---

## Main Configuration

```bash
sudo nano /etc/nginx/nginx.conf
```

### Optimized nginx.conf

```nginx
user www-data;
worker_processes auto;
pid /run/nginx.pid;
include /etc/nginx/modules-enabled/*.conf;

events {
    worker_connections 1024;
    use epoll;
    multi_accept on;
}

http {
    # Basic Settings
    sendfile on;
    tcp_nopush on;
    tcp_nodelay on;
    keepalive_timeout 65;
    types_hash_max_size 2048;
    server_tokens off;
    
    # Buffer sizes
    client_body_buffer_size 10K;
    client_header_buffer_size 1k;
    client_max_body_size 100M;
    large_client_header_buffers 4 32k;

    # Timeouts
    client_body_timeout 12;
    client_header_timeout 12;
    send_timeout 10;

    # MIME types
    include /etc/nginx/mime.types;
    default_type application/octet-stream;

    # Logging
    access_log /var/log/nginx/access.log;
    error_log /var/log/nginx/error.log;

    # Gzip Compression
    gzip on;
    gzip_vary on;
    gzip_proxied any;
    gzip_comp_level 6;
    gzip_types text/plain text/css text/xml application/json application/javascript application/rss+xml application/atom+xml image/svg+xml;

    # Rate limiting zone
    limit_req_zone $binary_remote_addr zone=one:10m rate=10r/s;

    # Security headers (can be overridden per server)
    add_header X-Frame-Options "SAMEORIGIN" always;
    add_header X-Content-Type-Options "nosniff" always;
    add_header X-XSS-Protection "1; mode=block" always;

    # Virtual Host Configs
    include /etc/nginx/conf.d/*.conf;
    include /etc/nginx/sites-enabled/*;
}
```

---

## Static Website Configuration

### Create Site Directory

```bash
sudo mkdir -p /var/www/example.com/html
sudo chown -R $USER:$USER /var/www/example.com
sudo chmod -R 755 /var/www/example.com
```

### Create Sample Page

```bash
nano /var/www/example.com/html/index.html
```

```html
<!DOCTYPE html>
<html>
<head>
    <title>Welcome to Example.com!</title>
</head>
<body>
    <h1>Success! Your Nginx server is working!</h1>
</body>
</html>
```

### Create Server Block

```bash
sudo nano /etc/nginx/sites-available/example.com
```

```nginx
server {
    listen 80;
    listen [::]:80;

    server_name example.com www.example.com;
    root /var/www/example.com/html;
    index index.html index.htm;

    # Logging
    access_log /var/log/nginx/example.com.access.log;
    error_log /var/log/nginx/example.com.error.log;

    # Main location
    location / {
        try_files $uri $uri/ =404;
    }

    # Cache static assets
    location ~* \.(jpg|jpeg|png|gif|ico|css|js|pdf|txt|woff|woff2)$ {
        expires 30d;
        add_header Cache-Control "public, immutable";
    }

    # Security - deny hidden files
    location ~ /\. {
        deny all;
    }
}
```

### Enable Site

```bash
sudo ln -s /etc/nginx/sites-available/example.com /etc/nginx/sites-enabled/
sudo nginx -t
sudo systemctl reload nginx
```

---

## Reverse Proxy Configuration

### Basic Reverse Proxy (Node.js App)

```nginx
server {
    listen 80;
    listen [::]:80;
    server_name api.example.com;

    location / {
        proxy_pass http://localhost:3000;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_cache_bypass $http_upgrade;
        proxy_read_timeout 86400;
    }
}
```

### WebSocket Support

```nginx
server {
    listen 80;
    server_name ws.example.com;

    location / {
        proxy_pass http://localhost:3001;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_read_timeout 3600;
        proxy_send_timeout 3600;
    }
}
```

### Multiple Apps on Subpaths

```nginx
server {
    listen 80;
    server_name example.com;

    # Main app
    location / {
        proxy_pass http://localhost:3000;
        proxy_http_version 1.1;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }

    # API
    location /api {
        proxy_pass http://localhost:4000;
        proxy_http_version 1.1;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }

    # Admin panel
    location /admin {
        proxy_pass http://localhost:5000;
        proxy_http_version 1.1;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }
}
```

---

## SSL/HTTPS Configuration

### With Certbot (Recommended)

```bash
sudo apt install certbot python3-certbot-nginx -y
sudo certbot --nginx -d example.com -d www.example.com
```

### Manual SSL Configuration

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

    # SSL certificates
    ssl_certificate /etc/letsencrypt/live/example.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/example.com/privkey.pem;

    # SSL settings
    ssl_session_timeout 1d;
    ssl_session_cache shared:SSL:50m;
    ssl_session_tickets off;

    # Modern configuration
    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_ciphers ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384;
    ssl_prefer_server_ciphers off;

    # HSTS
    add_header Strict-Transport-Security "max-age=63072000" always;

    # OCSP Stapling
    ssl_stapling on;
    ssl_stapling_verify on;
    ssl_trusted_certificate /etc/letsencrypt/live/example.com/chain.pem;
    resolver 8.8.8.8 8.8.4.4 valid=300s;

    root /var/www/example.com/html;
    index index.html;

    location / {
        try_files $uri $uri/ =404;
    }
}
```

---

## Load Balancing

### Round Robin (Default)

```nginx
upstream backend {
    server 127.0.0.1:3001;
    server 127.0.0.1:3002;
    server 127.0.0.1:3003;
}

server {
    listen 80;
    server_name example.com;

    location / {
        proxy_pass http://backend;
        proxy_http_version 1.1;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }
}
```

### Weighted Load Balancing

```nginx
upstream backend {
    server 127.0.0.1:3001 weight=3;
    server 127.0.0.1:3002 weight=2;
    server 127.0.0.1:3003 weight=1;
}
```

### Least Connections

```nginx
upstream backend {
    least_conn;
    server 127.0.0.1:3001;
    server 127.0.0.1:3002;
    server 127.0.0.1:3003;
}
```

### IP Hash (Sticky Sessions)

```nginx
upstream backend {
    ip_hash;
    server 127.0.0.1:3001;
    server 127.0.0.1:3002;
    server 127.0.0.1:3003;
}
```

### Health Checks

```nginx
upstream backend {
    server 127.0.0.1:3001 max_fails=3 fail_timeout=30s;
    server 127.0.0.1:3002 max_fails=3 fail_timeout=30s;
    server 127.0.0.1:3003 backup;  # Used only if others fail
}
```

---

## Rate Limiting

### Basic Rate Limiting

```nginx
# In http block (nginx.conf)
limit_req_zone $binary_remote_addr zone=api:10m rate=10r/s;

# In server block
server {
    location /api/ {
        limit_req zone=api burst=20 nodelay;
        proxy_pass http://localhost:3000;
    }
}
```

### Per-IP Connection Limit

```nginx
# In http block
limit_conn_zone $binary_remote_addr zone=addr:10m;

# In server block
server {
    limit_conn addr 10;  # 10 connections per IP
}
```

---

## Security Headers

```nginx
# Add to server block
add_header X-Frame-Options "SAMEORIGIN" always;
add_header X-Content-Type-Options "nosniff" always;
add_header X-XSS-Protection "1; mode=block" always;
add_header Referrer-Policy "strict-origin-when-cross-origin" always;
add_header Content-Security-Policy "default-src 'self' https: data: 'unsafe-inline' 'unsafe-eval'" always;
add_header Permissions-Policy "camera=(), microphone=(), geolocation=()" always;
```

---

## Caching

### Browser Caching

```nginx
location ~* \.(jpg|jpeg|png|gif|ico|css|js|pdf)$ {
    expires 30d;
    add_header Cache-Control "public, no-transform";
}
```

### Proxy Caching

```nginx
# In http block
proxy_cache_path /var/cache/nginx levels=1:2 keys_zone=my_cache:10m max_size=1g inactive=60m use_temp_path=off;

# In server block
server {
    location / {
        proxy_cache my_cache;
        proxy_cache_use_stale error timeout http_500 http_502 http_503 http_504;
        proxy_cache_valid 200 60m;
        proxy_cache_valid 404 1m;
        add_header X-Cache-Status $upstream_cache_status;
        proxy_pass http://localhost:3000;
    }
}
```

---

## Common Use Cases

### React/Vue/Angular SPA

```nginx
server {
    listen 80;
    server_name app.example.com;
    root /var/www/app/build;
    index index.html;

    location / {
        try_files $uri $uri/ /index.html;
    }

    location /api {
        proxy_pass http://localhost:3000;
        proxy_http_version 1.1;
        proxy_set_header Host $host;
    }
}
```

### PHP with FastCGI

```nginx
server {
    listen 80;
    server_name php.example.com;
    root /var/www/php-app;
    index index.php index.html;

    location / {
        try_files $uri $uri/ /index.php?$query_string;
    }

    location ~ \.php$ {
        include snippets/fastcgi-php.conf;
        fastcgi_pass unix:/var/run/php/php8.1-fpm.sock;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        include fastcgi_params;
    }

    location ~ /\.ht {
        deny all;
    }
}
```

### WordPress

```nginx
server {
    listen 80;
    server_name wordpress.example.com;
    root /var/www/wordpress;
    index index.php;

    client_max_body_size 64M;

    location / {
        try_files $uri $uri/ /index.php?$args;
    }

    location ~ \.php$ {
        include snippets/fastcgi-php.conf;
        fastcgi_pass unix:/var/run/php/php8.1-fpm.sock;
    }

    location ~* \.(js|css|png|jpg|jpeg|gif|ico|svg)$ {
        expires max;
        log_not_found off;
    }

    location ~ /\.ht {
        deny all;
    }

    location = /wp-config.php {
        deny all;
    }
}
```

---

## Troubleshooting

### Common Issues

```bash
# Test configuration
sudo nginx -t

# Check error logs
sudo tail -100 /var/log/nginx/error.log

# Check if nginx is running
sudo systemctl status nginx

# Check port 80/443
sudo ss -tlnp | grep -E ':80|:443'

# Check permissions
ls -la /var/www/example.com/
namei -l /var/www/example.com/html/index.html
```

### Permission Denied Errors

```bash
# Fix ownership
sudo chown -R www-data:www-data /var/www/example.com

# Fix SELinux (CentOS/RHEL)
sudo setsebool -P httpd_can_network_connect on
sudo chcon -Rt httpd_sys_content_t /var/www/example.com
```

### 502 Bad Gateway

```bash
# Check if backend is running
sudo systemctl status your-app

# Check if port is listening
sudo ss -tlnp | grep 3000

# Check nginx error log
sudo tail -f /var/log/nginx/error.log
```

---

## Quick Reference

| Task | Command |
|------|---------|
| Test config | `sudo nginx -t` |
| Reload | `sudo systemctl reload nginx` |
| Restart | `sudo systemctl restart nginx` |
| View errors | `sudo tail -f /var/log/nginx/error.log` |
| Enable site | `sudo ln -s /etc/nginx/sites-available/site /etc/nginx/sites-enabled/` |
| Disable site | `sudo rm /etc/nginx/sites-enabled/site` |

---

## Next Steps

1. **[SSL Setup](./certbot-installation.md)** - HTTPS with Let's Encrypt
2. **[Node.js Deployment](./deploy-nodejs-express.md)** - Deploy Node apps
3. **[Load Balancing](./nginx-load-balancing.md)** - Advanced load balancing

---

✅ Your Nginx server is ready!
