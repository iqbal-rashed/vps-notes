# Deploy a Node.js / Express App

This guide shows how to run a Node.js app on your server so it's always online, even after reboots.

**What we'll use:**
- **PM2** — Keeps your app running 24/7 and restarts it if it crashes
- **Nginx** — Receives web traffic and forwards it to your app

---

## Step 1 — Install Node.js

```bash
# Install Node.js 20 (LTS — stable version)
curl -fsSL https://deb.nodesource.com/setup_20.x | sudo -E bash -
sudo apt-get install -y nodejs

# Check it's installed
node -v
npm -v
```

---

## Step 2 — Install PM2

PM2 is a process manager. It keeps your app running and restarts it automatically.

```bash
sudo npm install -g pm2
```

---

## Step 3 — Upload Your App to the Server

**Option A: Clone from GitHub**
```bash
cd /var/www
git clone https://github.com/yourusername/your-repo.git myapp
cd myapp
```

**Option B: Upload files from your computer**
```bash
# Run this on YOUR LOCAL machine, not the server
scp -r ./your-app deployer@YOUR_SERVER_IP:/var/www/myapp
```

---

## Step 4 — Install Dependencies

```bash
cd /var/www/myapp
npm install --production
```

---

## Step 5 — Create Environment Variables File

Your app's secret settings go here:

```bash
nano /var/www/myapp/.env
```

```env
NODE_ENV=production
PORT=3000
DATABASE_URL=mongodb://localhost:27017/myapp
JWT_SECRET=change-this-to-a-long-random-string
```

Secure the file so only your user can read it:
```bash
chmod 600 /var/www/myapp/.env
```

---

## Step 6 — Start Your App with PM2

```bash
cd /var/www/myapp

# Start your app
pm2 start app.js --name myapp

# Save the process list (so it restarts after reboot)
pm2 save

# Set up PM2 to start on boot
pm2 startup
# Copy and run the command it gives you
```

---

## Step 7 — Configure Nginx

Create an Nginx config that forwards web traffic to your app:

```bash
sudo nano /etc/nginx/sites-available/myapp
```

```nginx
server {
    listen 80;
    server_name example.com www.example.com;

    # Security headers
    add_header X-Content-Type-Options "nosniff" always;
    add_header X-Frame-Options "SAMEORIGIN" always;
    add_header X-XSS-Protection "1; mode=block" always;

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
sudo ln -s /etc/nginx/sites-available/myapp /etc/nginx/sites-enabled/
sudo nginx -t
sudo systemctl reload nginx
```

---

## Step 8 — Add SSL (HTTPS)

```bash
sudo certbot --nginx -d example.com -d www.example.com
```

---

## PM2 Commands You'll Use Often

```bash
# See all running apps
pm2 list

# See logs
pm2 logs myapp

# Restart app
pm2 restart myapp

# Reload app (no downtime — use this instead of restart)
pm2 reload myapp

# Stop app
pm2 stop myapp

# Live monitoring
pm2 monit
```

---

## Deploy Updates

When you push new code, run this to update your server:

```bash
cd /var/www/myapp
git pull origin main
npm install --production
pm2 reload myapp
```

Or create a script `deploy.sh`:

```bash
#!/bin/bash
set -e

cd /var/www/myapp
git pull origin main
npm install --production
npm run build   # Remove this line if you don't have a build step
pm2 reload myapp

echo "Deployment done!"
```

```bash
chmod +x deploy.sh
./deploy.sh
```

---

## Troubleshooting

**App crashed / not running:**
```bash
pm2 logs myapp --lines 50
```

**502 Bad Gateway in browser:**
```bash
# Check if your app is actually running
pm2 status

# Check what port it's on
sudo ss -tlnp | grep node
```

**App not starting after reboot:**
```bash
pm2 startup
# Run the command it gives you
pm2 save
```

---

## Next Steps

1. [SSL Certificate](./certbot-installation.md) — Add HTTPS
2. [GitHub Actions](./github-actions-deploy.md) — Auto-deploy on git push
3. [Monitoring](./server-monitoring-guide.md) — Watch your server health
