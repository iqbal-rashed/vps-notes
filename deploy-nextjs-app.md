# Deploy a Next.js App (Server-Side)

Next.js runs as a real server on your VPS. This means:
- Server-side rendering (SSR) works
- API routes work
- Dynamic pages work
- PM2 keeps it running 24/7

> This guide uses **standalone mode** which is the best way to deploy Next.js on a VPS.

---

## Step 1 — Prepare Your Next.js App for Deployment

In your **next.config.js** (or `next.config.ts`), add `output: 'standalone'`:

```javascript
// next.config.js
const nextConfig = {
  output: 'standalone',
};

module.exports = nextConfig;
```

This makes Next.js bundle everything it needs into one folder — easier to deploy.

---

## Step 2 — Install Node.js and PM2 on the Server

If you haven't already:

```bash
# Install Node.js 20
curl -fsSL https://deb.nodesource.com/setup_20.x | sudo -E bash -
sudo apt-get install -y nodejs

# Install PM2
sudo npm install -g pm2
```

---

## Step 3 — Upload Your App to the Server

**Option A: Clone from GitHub (recommended)**
```bash
cd /var/www
git clone https://github.com/yourusername/your-nextjs-app.git nextjs-app
cd nextjs-app
```

**Option B: Upload from local machine**
```bash
# Run this on your LOCAL machine
scp -r ./your-nextjs-app deployer@YOUR_SERVER_IP:/var/www/nextjs-app
```

---

## Step 4 — Set Up Environment Variables

```bash
nano /var/www/nextjs-app/.env.production
```

```env
NODE_ENV=production
NEXT_PUBLIC_API_URL=https://api.example.com
DATABASE_URL=mongodb://localhost:27017/myapp
NEXTAUTH_SECRET=change-this-to-a-long-random-string
NEXTAUTH_URL=https://example.com
```

Secure it:
```bash
chmod 600 /var/www/nextjs-app/.env.production
```

---

## Step 5 — Install and Build

```bash
cd /var/www/nextjs-app
npm install
npm run build
```

After the build, Next.js creates a `.next/standalone/` folder with everything needed to run.

---

## Step 6 — Copy the Standalone Build

```bash
# Create production folder
mkdir -p /var/www/nextjs-production

# Copy the standalone output
cp -r /var/www/nextjs-app/.next/standalone/. /var/www/nextjs-production/

# Copy static files (CSS, JS, images)
cp -r /var/www/nextjs-app/.next/static /var/www/nextjs-production/.next/static

# Copy public folder (favicon, images, etc.)
cp -r /var/www/nextjs-app/public /var/www/nextjs-production/public
```

---

## Step 7 — Start with PM2

Create a PM2 config file:

```bash
nano /var/www/nextjs-production/ecosystem.config.js
```

```javascript
module.exports = {
  apps: [{
    name: 'nextjs-app',
    script: 'server.js',
    cwd: '/var/www/nextjs-production',
    env: {
      NODE_ENV: 'production',
      PORT: 3000
    },
    max_memory_restart: '500M',
    error_file: '/var/log/pm2/nextjs-error.log',
    out_file: '/var/log/pm2/nextjs-out.log',
  }]
};
```

Start it:
```bash
cd /var/www/nextjs-production
pm2 start ecosystem.config.js
pm2 save
pm2 startup
# Run the command it gives you
```

Check it's running:
```bash
pm2 status
pm2 logs nextjs-app
```

---

## Step 8 — Configure Nginx

```bash
sudo nano /etc/nginx/sites-available/nextjs-app
```

```nginx
server {
    listen 80;
    server_name example.com www.example.com;

    # Security headers
    add_header X-Content-Type-Options "nosniff" always;
    add_header X-Frame-Options "SAMEORIGIN" always;
    add_header X-XSS-Protection "1; mode=block" always;

    # Next.js static files — cache for 1 year
    location /_next/static/ {
        proxy_pass http://localhost:3000;
        expires 365d;
        add_header Cache-Control "public, immutable";
        access_log off;
    }

    # API routes — no caching
    location /api/ {
        proxy_pass http://localhost:3000;
        proxy_http_version 1.1;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }

    # All other pages — forward to Next.js server
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

Enable the site:
```bash
sudo ln -s /etc/nginx/sites-available/nextjs-app /etc/nginx/sites-enabled/
sudo nginx -t
sudo systemctl reload nginx
```

---

## Step 9 — Add SSL (HTTPS)

```bash
sudo certbot --nginx -d example.com -d www.example.com
```

---

## Deploy Updates (When You Push New Code)

Create a deploy script:

```bash
nano /var/www/nextjs-app/deploy.sh
```

```bash
#!/bin/bash
set -e

APP_DIR="/var/www/nextjs-app"
PROD_DIR="/var/www/nextjs-production"

echo "Pulling latest code..."
cd $APP_DIR
git pull origin main

echo "Installing dependencies..."
npm install

echo "Building..."
npm run build

echo "Copying build..."
cp -r .next/standalone/. $PROD_DIR/
cp -r .next/static $PROD_DIR/.next/
cp -r public $PROD_DIR/

echo "Reloading app..."
pm2 reload nextjs-app

echo "Done!"
```

```bash
chmod +x /var/www/nextjs-app/deploy.sh
```

To deploy:
```bash
/var/www/nextjs-app/deploy.sh
```

---

## PM2 Commands

```bash
# See running apps
pm2 list

# See logs
pm2 logs nextjs-app

# Reload (no downtime)
pm2 reload nextjs-app

# Restart
pm2 restart nextjs-app
```

---

## Troubleshooting

**Build fails:**
```bash
# If memory error, increase Node.js memory limit
NODE_OPTIONS="--max-old-space-size=4096" npm run build
```

**App crashes at startup:**
```bash
pm2 logs nextjs-app --lines 100
```

**502 Bad Gateway:**
```bash
# Check Next.js is running
pm2 status

# Check it's listening on port 3000
sudo ss -tlnp | grep 3000
```

**Pages not loading after deploy:**
```bash
# Sometimes a full restart is needed
pm2 restart nextjs-app
```

---

## Next Steps

1. [SSL Certificate](./certbot-installation.md) — Add HTTPS
2. [GitHub Actions](./github-actions-deploy.md) — Auto-deploy on git push
3. [Monitoring](./server-monitoring-guide.md) — Monitor your server
