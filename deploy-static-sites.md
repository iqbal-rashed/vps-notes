# Deploy a Static Site (React, Vue, Angular)

Static sites are HTML/CSS/JS files — no server needed. You just build your app and serve the files with Nginx.

---

## Step 1 — Build Your App Locally

```bash
# React (Vite or Create React App)
npm run build
# Output: dist/ or build/

# Vue
npm run build
# Output: dist/

# Angular
ng build
# Output: dist/your-app-name/
```

---

## Step 2 — Upload Files to the Server

**Option A: rsync (fast, only uploads changed files)**
```bash
rsync -avz ./dist/ deployer@YOUR_SERVER_IP:/var/www/mysite/
```

**Option B: scp (simple copy)**
```bash
scp -r ./dist/* deployer@YOUR_SERVER_IP:/var/www/mysite/
```

**Option C: Git (if your built files are in git)**
```bash
# On the server
cd /var/www/mysite
git pull origin main
```

---

## Step 3 — Create Nginx Config

```bash
sudo nano /etc/nginx/sites-available/mysite
```

```nginx
server {
    listen 80;
    server_name example.com www.example.com;
    root /var/www/mysite;
    index index.html;

    # This makes React/Vue routing work
    # Without it, refreshing on /about gives a 404
    location / {
        try_files $uri $uri/ /index.html;
    }

    # Cache images, CSS, JS for 1 year
    location ~* \.(js|css|png|jpg|jpeg|gif|ico|svg|woff|woff2)$ {
        expires 1y;
        add_header Cache-Control "public, immutable";
    }

    # Security headers
    add_header X-Content-Type-Options "nosniff" always;
    add_header X-Frame-Options "SAMEORIGIN" always;

    # Block hidden files
    location ~ /\. {
        deny all;
    }
}
```

Enable it:
```bash
sudo ln -s /etc/nginx/sites-available/mysite /etc/nginx/sites-enabled/
sudo nginx -t
sudo systemctl reload nginx
```

---

## Step 4 — Add SSL (HTTPS)

```bash
sudo certbot --nginx -d example.com -d www.example.com
```

---

## Calling an API from Your Static Site

If your frontend needs to talk to a backend API, proxy it through Nginx. Add this to your server block:

```nginx
# Forward /api requests to your Node.js backend
location /api/ {
    proxy_pass http://localhost:3000;
    proxy_set_header Host $host;
    proxy_set_header X-Real-IP $remote_addr;
}
```

This avoids CORS issues since both frontend and API are on the same domain.

---

## Auto-Deploy on Every Build (GitHub Actions)

```yaml
# .github/workflows/deploy.yml
name: Deploy Static Site

on:
  push:
    branches: [main]

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'npm'

      - name: Build
        run: |
          npm ci
          npm run build

      - name: Upload to server
        uses: appleboy/scp-action@v0.1.7
        with:
          host: ${{ secrets.HOST }}
          username: ${{ secrets.USERNAME }}
          key: ${{ secrets.SSH_PRIVATE_KEY }}
          port: ${{ secrets.PORT || 22 }}
          source: "dist/*"
          target: "/var/www/mysite/"
          strip_components: 1
```

---

## Troubleshooting

**Page works on `/` but refreshing `/about` gives 404:**
Make sure your Nginx config has `try_files $uri $uri/ /index.html;`

**CSS/JS not loading:**
```bash
# Check file permissions
ls -la /var/www/mysite/
sudo chown -R www-data:www-data /var/www/mysite
```

**Old version still showing:**
Your browser is probably caching. Force-refresh with `Ctrl+Shift+R` or clear cache.
