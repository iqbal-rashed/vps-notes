# Deploy Node.js/Express Application

Complete guide for deploying Node.js applications with PM2, Nginx reverse proxy, and production best practices.

---

## Prerequisites

- Ubuntu/Debian VPS with sudo access
- Domain name pointing to your server
- Application code ready to deploy

---

## 1. Install Node.js

### Using NodeSource (Recommended)

```bash
# Node.js 20 LTS (Recommended)
curl -fsSL https://deb.nodesource.com/setup_20.x | sudo -E bash -
sudo apt-get install -y nodejs

# Verify installation
node -v
npm -v
```

### Using NVM (Multiple Versions)

```bash
# Install NVM
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.7/install.sh | bash

# Reload shell
source ~/.bashrc

# Install Node.js
nvm install 20
nvm use 20
nvm alias default 20

# Verify
node -v
```

---

## 2. Install Package Managers & PM2

```bash
# Install Yarn (optional)
sudo npm install -g yarn

# Install pnpm (optional)
sudo npm install -g pnpm

# Install PM2 (Process Manager)
sudo npm install -g pm2
```

---

## 3. Create Deploy User (Recommended)

```bash
# Create user
sudo adduser deployer
sudo usermod -aG sudo deployer

# Switch to deployer
su - deployer
```

---

## 4. Clone Your Application

```bash
# Create app directory
sudo mkdir -p /var/www/myapp
sudo chown -R deployer:deployer /var/www/myapp

# Clone repository
cd /var/www/myapp
git clone https://github.com/yourusername/your-repo.git .

# Or upload via SCP
scp -r ./your-app deployer@server-ip:/var/www/myapp
```

---

## 5. Install Dependencies

```bash
cd /var/www/myapp

# Using npm
npm install --production

# Using yarn
yarn install --production

# Using pnpm
pnpm install --prod
```

---

## 6. Environment Variables

### Create .env file

```bash
nano /var/www/myapp/.env
```

```env
NODE_ENV=production
PORT=3000
DATABASE_URL=mongodb://localhost:27017/myapp
JWT_SECRET=your-super-secret-key
API_KEY=your-api-key
```

### Secure the file

```bash
chmod 600 /var/www/myapp/.env
```

---

## 7. PM2 Process Manager

### Start Application

```bash
cd /var/www/myapp

# Basic start
pm2 start app.js --name myapp

# With npm script
pm2 start npm --name myapp -- start

# With environment file
pm2 start app.js --name myapp --env production
```

### PM2 Ecosystem File (Recommended)

Create `ecosystem.config.js`:

```javascript
module.exports = {
  apps: [{
    name: 'myapp',
    script: './app.js',
    // Or for npm start:
    // script: 'npm',
    // args: 'start',
    instances: 'max',        // Use all CPU cores
    exec_mode: 'cluster',    // Cluster mode
    watch: false,            // Don't watch in production
    max_memory_restart: '1G',
    env: {
      NODE_ENV: 'development'
    },
    env_production: {
      NODE_ENV: 'production',
      PORT: 3000
    },
    // Logging
    error_file: '/var/log/pm2/myapp-error.log',
    out_file: '/var/log/pm2/myapp-out.log',
    log_file: '/var/log/pm2/myapp-combined.log',
    time: true,
    // Restart policy
    restart_delay: 4000,
    autorestart: true,
    max_restarts: 10,
    min_uptime: '10s'
  }]
};
```

Start with ecosystem:

```bash
pm2 start ecosystem.config.js --env production
```

### PM2 Commands

```bash
# Process management
pm2 start myapp
pm2 stop myapp
pm2 restart myapp
pm2 reload myapp    # Zero-downtime reload
pm2 delete myapp

# View status
pm2 status
pm2 list
pm2 show myapp
pm2 monit          # Real-time monitoring

# Logs
pm2 logs
pm2 logs myapp
pm2 logs myapp --lines 100
pm2 flush          # Clear all logs

# Save process list (for startup)
pm2 save

# Startup script
pm2 startup
# Run the command it outputs, then:
pm2 save
```

### Auto-Start on Reboot

```bash
# Generate startup script
pm2 startup systemd

# Copy and run the command it outputs
sudo env PATH=$PATH:/usr/bin pm2 startup systemd -u deployer --hp /home/deployer

# Save current process list
pm2 save
```

---

## 8. Install and Configure Nginx

```bash
sudo apt install nginx -y
sudo systemctl enable nginx
```

### Create Server Block

```bash
sudo nano /etc/nginx/sites-available/myapp
```

```nginx
server {
    listen 80;
    listen [::]:80;
    server_name myapp.com www.myapp.com;

    # Logging
    access_log /var/log/nginx/myapp.access.log;
    error_log /var/log/nginx/myapp.error.log;

    # Security headers
    add_header X-Frame-Options "SAMEORIGIN" always;
    add_header X-Content-Type-Options "nosniff" always;
    add_header X-XSS-Protection "1; mode=block" always;

    # Gzip
    gzip on;
    gzip_types text/plain application/json application/javascript text/css;

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

    # Static files (if serving from Node)
    location /static/ {
        alias /var/www/myapp/public/;
        expires 30d;
        add_header Cache-Control "public, immutable";
    }

    # Health check endpoint
    location /health {
        proxy_pass http://localhost:3000/health;
        access_log off;
    }
}
```

### Enable Site

```bash
sudo ln -s /etc/nginx/sites-available/myapp /etc/nginx/sites-enabled/
sudo nginx -t
sudo systemctl reload nginx
```

---

## 9. SSL with Certbot

```bash
sudo apt install certbot python3-certbot-nginx -y
sudo certbot --nginx -d myapp.com -d www.myapp.com
```

---

## 10. Firewall Configuration

```bash
sudo ufw allow 'Nginx Full'
sudo ufw allow ssh
sudo ufw enable
```

---

## 11. Deployment Script

Create `deploy.sh`:

```bash
#!/bin/bash
set -e

APP_DIR="/var/www/myapp"
BRANCH="main"

echo "🚀 Starting deployment..."

cd $APP_DIR

echo "📥 Pulling latest changes..."
git fetch origin
git reset --hard origin/$BRANCH

echo "📦 Installing dependencies..."
npm ci --production

echo "🔨 Building application..."
npm run build

echo "🔄 Reloading PM2..."
pm2 reload myapp

echo "✅ Deployment complete!"
```

```bash
chmod +x deploy.sh
./deploy.sh
```

---

## 12. Health Check Endpoint

Add to your Express app:

```javascript
app.get('/health', (req, res) => {
  res.status(200).json({
    status: 'healthy',
    timestamp: new Date().toISOString(),
    uptime: process.uptime()
  });
});
```

---

## 13. Logging Best Practices

### Using Winston

```javascript
const winston = require('winston');

const logger = winston.createLogger({
  level: 'info',
  format: winston.format.combine(
    winston.format.timestamp(),
    winston.format.json()
  ),
  transports: [
    new winston.transports.File({ filename: 'logs/error.log', level: 'error' }),
    new winston.transports.File({ filename: 'logs/combined.log' })
  ]
});

if (process.env.NODE_ENV !== 'production') {
  logger.add(new winston.transports.Console({
    format: winston.format.simple()
  }));
}
```

### Log Rotation with PM2

```javascript
// In ecosystem.config.js
{
  log_date_format: 'YYYY-MM-DD HH:mm Z',
  merge_logs: true,
  error_file: '/var/log/pm2/error.log',
  out_file: '/var/log/pm2/out.log'
}
```

Install log rotation:
```bash
pm2 install pm2-logrotate
pm2 set pm2-logrotate:max_size 10M
pm2 set pm2-logrotate:retain 7
```

---

## 14. Common Issues

### EACCES Permission Denied

```bash
# Fix npm permissions
mkdir ~/.npm-global
npm config set prefix '~/.npm-global'
echo 'export PATH=~/.npm-global/bin:$PATH' >> ~/.bashrc
source ~/.bashrc
```

### Port Already in Use

```bash
# Find process using port
sudo lsof -i :3000
sudo kill -9 <PID>

# Or use PM2 to manage
pm2 delete all
pm2 start ecosystem.config.js
```

### Node Process Crashes

```bash
# Check PM2 logs
pm2 logs myapp --lines 200

# Check system logs
sudo journalctl -u nginx -f
```

### Memory Issues

```bash
# Increase Node memory limit
node --max-old-space-size=4096 app.js

# In PM2 ecosystem
{
  node_args: '--max-old-space-size=4096',
  max_memory_restart: '1G'
}
```

---

## Quick Reference

| Task | Command |
|------|---------|
| Start app | `pm2 start app.js --name myapp` |
| Stop app | `pm2 stop myapp` |
| Restart app | `pm2 restart myapp` |
| Zero-downtime reload | `pm2 reload myapp` |
| View logs | `pm2 logs myapp` |
| Monitor | `pm2 monit` |
| Save processes | `pm2 save` |
| List processes | `pm2 list` |

---

## Project Structure (Recommended)

```
/var/www/myapp/
├── app.js                 # Entry point
├── ecosystem.config.js    # PM2 config
├── package.json
├── .env                   # Environment variables
├── .gitignore
├── src/                   # Source code
├── public/                # Static files
├── logs/                  # Application logs
└── deploy.sh              # Deployment script
```

---

## Next Steps

1. **[PM2 Advanced Guide](./pm2-advanced-guide.md)** - Clustering and monitoring
2. **[SSL Setup](./certbot-installation.md)** - HTTPS with Certbot
3. **[CI/CD](./github-actions-deploy.md)** - Automated deployments
4. **[Monitoring](./server-monitoring-guide.md)** - Application monitoring

---

✅ Your Node.js application is now deployed and running in production!
