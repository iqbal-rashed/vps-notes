# Environment Variables Guide

Managing secrets and environment variables in production.

---

## 1. Using .env Files

### Create .env

```bash
nano /var/www/myapp/.env
```

```env
NODE_ENV=production
PORT=3000
DATABASE_URL=mongodb://localhost:27017/myapp
JWT_SECRET=your-super-secret-key
API_KEY=abc123
```

### Secure Permissions

```bash
chmod 600 .env
chown deployer:deployer .env
```

---

## 2. Node.js

### Using dotenv

```bash
npm install dotenv
```

```javascript
// Load at entry point
require('dotenv').config();

// Access
console.log(process.env.DATABASE_URL);
```

### PM2 with .env

```javascript
// ecosystem.config.js
module.exports = {
  apps: [{
    name: 'myapp',
    script: 'app.js',
    env_file: '.env',
    // Or inline:
    env: {
      NODE_ENV: 'production'
    }
  }]
};
```

---

## 3. Docker

### docker-compose.yml

```yaml
services:
  app:
    env_file:
      - .env
    environment:
      - NODE_ENV=production
```

### Command line

```bash
docker run --env-file .env myimage
docker run -e NODE_ENV=production myimage
```

---

## 4. Systemd Service

```ini
[Service]
Environment="NODE_ENV=production"
Environment="PORT=3000"
EnvironmentFile=/var/www/myapp/.env
```

---

## 5. GitHub Actions Secrets

```yaml
- name: Deploy
  env:
    DATABASE_URL: ${{ secrets.DATABASE_URL }}
  run: |
    echo "DATABASE_URL=$DATABASE_URL" >> .env
```

---

## 6. Never Commit Secrets

### .gitignore

```
.env
.env.local
.env.production
*.pem
```

---

## Best Practices

- ✅ Use `.env` files for local/server config
- ✅ Never commit secrets to Git
- ✅ Use different keys per environment
- ✅ Rotate secrets regularly
- ✅ Use secret managers for teams (Vault, AWS Secrets Manager)
- ✅ Restrict file permissions (600)

---

✅ Environment configured!
