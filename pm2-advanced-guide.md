# PM2 Advanced Guide

Complete guide for PM2 process manager with cluster mode, monitoring, and logs.

---

## 1. Install PM2

```bash
npm install -g pm2
```

---

## 2. Basic Commands

```bash
# Start application
pm2 start app.js --name myapp

# With npm script
pm2 start npm --name myapp -- start

# List processes
pm2 list
pm2 status

# Stop/Restart/Reload
pm2 stop myapp
pm2 restart myapp
pm2 reload myapp    # Zero-downtime

# Delete process
pm2 delete myapp

# Delete all
pm2 delete all
```

---

## 3. Ecosystem File

Create `ecosystem.config.js`:

```javascript
module.exports = {
  apps: [{
    name: 'myapp',
    script: './dist/index.js',
    instances: 'max',           // Use all CPUs
    exec_mode: 'cluster',
    watch: false,
    max_memory_restart: '1G',
    env: {
      NODE_ENV: 'development'
    },
    env_production: {
      NODE_ENV: 'production',
      PORT: 3000
    },
    // Logs
    error_file: './logs/error.log',
    out_file: './logs/out.log',
    log_date_format: 'YYYY-MM-DD HH:mm:ss',
    merge_logs: true,
    // Restart behavior
    autorestart: true,
    max_restarts: 10,
    min_uptime: '10s',
    restart_delay: 4000
  }]
};
```

Start with ecosystem:
```bash
pm2 start ecosystem.config.js --env production
```

---

## 4. Cluster Mode

```javascript
// ecosystem.config.js
{
  instances: 'max',        // All CPUs
  instances: 4,            // Specific number
  exec_mode: 'cluster'     // Enable clustering
}
```

Manage clusters:
```bash
pm2 scale myapp 4    # Scale to 4 instances
pm2 scale myapp +2   # Add 2 instances
pm2 scale myapp -1   # Remove 1 instance
```

---

## 5. Monitoring

```bash
# Real-time dashboard
pm2 monit

# Process details
pm2 show myapp
pm2 describe myapp

# CPU/Memory stats
pm2 status

# Environment variables
pm2 env myapp
```

---

## 6. Logs

```bash
# View logs
pm2 logs
pm2 logs myapp
pm2 logs myapp --lines 100

# Follow logs
pm2 logs --follow

# Clear logs
pm2 flush

# Install log rotation
pm2 install pm2-logrotate
pm2 set pm2-logrotate:max_size 10M
pm2 set pm2-logrotate:retain 7
pm2 set pm2-logrotate:compress true
```

---

## 7. Startup Script

```bash
# Generate startup script
pm2 startup

# Run the command it outputs, then save:
pm2 save

# Remove startup
pm2 unstartup
```

---

## 8. Deploy Command

```javascript
// ecosystem.config.js
module.exports = {
  apps: [{ /* app config */ }],
  deploy: {
    production: {
      user: 'deployer',
      host: 'server-ip',
      ref: 'origin/main',
      repo: 'git@github.com:user/repo.git',
      path: '/var/www/myapp',
      'post-deploy': 'npm ci && npm run build && pm2 reload ecosystem.config.js --env production'
    }
  }
};
```

```bash
# First time setup
pm2 deploy production setup

# Deploy
pm2 deploy production
```

---

## 9. Multiple Apps

```javascript
module.exports = {
  apps: [
    {
      name: 'api',
      script: './api/index.js',
      instances: 2
    },
    {
      name: 'worker',
      script: './worker/index.js',
      instances: 1
    }
  ]
};
```

---

## Quick Reference

| Task | Command |
|------|---------|
| Start | `pm2 start app.js` |
| Stop | `pm2 stop app` |
| Restart | `pm2 restart app` |
| Zero-downtime | `pm2 reload app` |
| Logs | `pm2 logs app` |
| Monitor | `pm2 monit` |
| Save | `pm2 save` |
| Startup | `pm2 startup` |

---

✅ PM2 is configured!
