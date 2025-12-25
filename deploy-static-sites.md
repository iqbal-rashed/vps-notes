# Deploy Static Sites

Guide for deploying static HTML, React, Vue, Angular builds.

---

## 1. Build Your App

### React (Create React App / Vite)

```bash
npm run build
# Output: build/ or dist/
```

### Vue

```bash
npm run build
# Output: dist/
```

### Angular

```bash
ng build --prod
# Output: dist/project-name/
```

---

## 2. Upload Files

### Using rsync

```bash
rsync -avz ./dist/ user@server:/var/www/mysite/
```

### Using scp

```bash
scp -r ./dist/* user@server:/var/www/mysite/
```

### Using Git

```bash
ssh user@server
cd /var/www/mysite
git pull origin main
```

---

## 3. Nginx Configuration

```bash
sudo nano /etc/nginx/sites-available/mysite
```

```nginx
server {
    listen 80;
    server_name example.com www.example.com;
    root /var/www/mysite;
    index index.html;

    # SPA routing
    location / {
        try_files $uri $uri/ /index.html;
    }

    # Cache static assets
    location ~* \.(js|css|png|jpg|jpeg|gif|ico|svg|woff|woff2)$ {
        expires 1y;
        add_header Cache-Control "public, immutable";
    }

    # Security
    location ~ /\. {
        deny all;
    }
}
```

```bash
sudo ln -s /etc/nginx/sites-available/mysite /etc/nginx/sites-enabled/
sudo nginx -t
sudo systemctl reload nginx
```

---

## 4. SSL Certificate

```bash
sudo certbot --nginx -d example.com -d www.example.com
```

---

## 5. GitHub Actions Deploy

```yaml
name: Deploy Static Site

on:
  push:
    branches: [main]

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: '20'
      
      - run: npm ci
      - run: npm run build
      
      - name: Deploy
        uses: appleboy/scp-action@v0.1.7
        with:
          host: ${{ secrets.HOST }}
          username: ${{ secrets.USERNAME }}
          key: ${{ secrets.SSH_KEY }}
          source: "dist/*"
          target: "/var/www/mysite"
          strip_components: 1
```

---

## Quick Reference

| Framework | Build Command | Output |
|-----------|---------------|--------|
| React | `npm run build` | build/ or dist/ |
| Vue | `npm run build` | dist/ |
| Angular | `ng build` | dist/ |
| Next.js SSG | `npm run build && npm run export` | out/ |

---

✅ Static site deployed!
