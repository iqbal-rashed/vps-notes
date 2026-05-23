# PM2 Process Manager

PM2 keeps your Node.js app running 24/7. If the app crashes, PM2 restarts it. If the server reboots, PM2 starts your app again automatically.

---

## Install PM2

```bash
sudo npm install -g pm2
```

---

## Start Your App

```bash
# Basic start
pm2 start app.js --name myapp

# If your app uses `npm start`
pm2 start npm --name myapp -- start

# See all running apps
pm2 list
```

---

## The Ecosystem Config File (Recommended)

Instead of typing start options every time, put them in a config file.

Create `ecosystem.config.js` in your app folder:

```javascript
module.exports = {
  apps: [{
    name: 'myapp',
    script: './app.js',         // Your entry file
    instances: 1,               // Use 'max' to use all CPU cores
    watch: false,               // Don't restart on file changes (use false in production)
    max_memory_restart: '500M', // Restart if app uses more than 500MB RAM
    env: {
      NODE_ENV: 'production',
      PORT: 3000
    },
    error_file: '/var/log/pm2/myapp-error.log',
    out_file: '/var/log/pm2/myapp-out.log',
    autorestart: true,
    restart_delay: 3000         // Wait 3 seconds before restarting
  }]
};
```

Start with the config:
```bash
pm2 start ecosystem.config.js
```

---

## Auto-Start on Server Reboot

Do this once after your app is running:

```bash
# Set up PM2 to start on boot
pm2 startup

# Run the command it gives you (something like):
# sudo env PATH=$PATH:/usr/bin pm2 startup systemd -u deployer --hp /home/deployer

# Save your current apps
pm2 save
```

Now your apps will start automatically after every server reboot.

---

## Common PM2 Commands

```bash
# See all apps and their status
pm2 list

# See live logs
pm2 logs myapp

# See last 100 lines of logs
pm2 logs myapp --lines 100

# Reload app (no downtime — use this for deploys)
pm2 reload myapp

# Restart app (brief downtime)
pm2 restart myapp

# Stop app
pm2 stop myapp

# Remove app from PM2
pm2 delete myapp

# Live monitoring (CPU, memory, logs)
pm2 monit

# Save current app list
pm2 save

# Clear all logs
pm2 flush
```

---

## Multiple Apps

You can run multiple apps in one ecosystem file:

```javascript
module.exports = {
  apps: [
    {
      name: 'frontend',
      script: './server.js',
      env: { PORT: 3000, NODE_ENV: 'production' }
    },
    {
      name: 'api',
      script: './api/server.js',
      env: { PORT: 4000, NODE_ENV: 'production' }
    }
  ]
};
```

---

## Log Rotation (Prevent Huge Log Files)

Install the log rotation plugin:

```bash
pm2 install pm2-logrotate

# Max log file size: 10MB
pm2 set pm2-logrotate:max_size 10M

# Keep 7 days of logs
pm2 set pm2-logrotate:retain 7
```

---

## Troubleshooting

**App keeps crashing:**
```bash
pm2 logs myapp --lines 200
```

**App not starting after reboot:**
```bash
# Re-run startup setup
pm2 startup
# Copy and run the command it gives you
pm2 save
```

**Port already in use:**
```bash
sudo lsof -i :3000
# Kill the process using that port
sudo kill -9 <PID>
```

---

## Quick Reference

| Task | Command |
|------|---------|
| Start | `pm2 start app.js --name myapp` |
| Reload (no downtime) | `pm2 reload myapp` |
| Logs | `pm2 logs myapp` |
| Status | `pm2 list` |
| Monitor | `pm2 monit` |
| Save list | `pm2 save` |
