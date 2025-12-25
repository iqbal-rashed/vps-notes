# Deploy Next.js Application

Complete guide for deploying Next.js applications with SSR/SSG support on VPS.

---

## Prerequisites

- Node.js 18+ installed
- PM2 process manager
- Nginx web server
- Domain pointing to your server

---

## 1. Build Modes

### Static Export (SSG Only)

For fully static sites:

```javascript
// next.config.js
/** @type {import('next').NextConfig} */
const nextConfig = {
  output: 'export',
  trailingSlash: true,
  images: {
    unoptimized: true,
  },
};

module.exports = nextConfig;
```

```bash
npm run build
# Output in 'out' directory
```

### Standalone Mode (Recommended for VPS)

For SSR and full features:

```javascript
// next.config.js
/** @type {import('next').NextConfig} */
const nextConfig = {
  output: 'standalone',
};

module.exports = nextConfig;
```

---

## 2. Prepare Application

### Clone and Install

```bash
cd /var/www
git clone https://github.com/yourusername/nextjs-app.git
cd nextjs-app
npm install
```

### Environment Variables

```bash
nano .env.production
```

```env
NODE_ENV=production
NEXT_PUBLIC_API_URL=https://api.example.com
DATABASE_URL=mongodb://localhost:27017/myapp
NEXTAUTH_SECRET=your-secret-key
NEXTAUTH_URL=https://example.com
```

### Build Application

```bash
npm run build
```

---

## 3. Deploy with Standalone Output

### Copy Standalone Build

```bash
# After build, the standalone output is in .next/standalone
cp -r .next/standalone /var/www/nextjs-production
cp -r .next/static /var/www/nextjs-production/.next/static
cp -r public /var/www/nextjs-production/public
```

### Create Ecosystem File

```bash
nano /var/www/nextjs-production/ecosystem.config.js
```

```javascript
module.exports = {
  apps: [{
    name: 'nextjs-app',
    script: 'server.js',
    cwd: '/var/www/nextjs-production',
    instances: 'max',
    exec_mode: 'cluster',
    env: {
      NODE_ENV: 'production',
      PORT: 3000
    },
    max_memory_restart: '500M',
    error_file: '/var/log/pm2/nextjs-error.log',
    out_file: '/var/log/pm2/nextjs-out.log',
    time: true
  }]
};
```

### Start with PM2

```bash
cd /var/www/nextjs-production
pm2 start ecosystem.config.js
pm2 save
pm2 startup
```

---

## 4. Deploy with npm start

Alternative without standalone:

```bash
nano /var/www/nextjs-app/ecosystem.config.js
```

```javascript
module.exports = {
  apps: [{
    name: 'nextjs-app',
    script: 'npm',
    args: 'start',
    cwd: '/var/www/nextjs-app',
    instances: 1,  // Next.js handles its own clustering
    env: {
      NODE_ENV: 'production',
      PORT: 3000
    }
  }]
};
```

---

## 5. Nginx Configuration

### Basic Reverse Proxy

```bash
sudo nano /etc/nginx/sites-available/nextjs-app
```

```nginx
server {
    listen 80;
    listen [::]:80;
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
        proxy_cache_bypass $http_upgrade;
    }
}
```

### Optimized Configuration with Caching

```nginx
# Upstream
upstream nextjs_upstream {
    server 127.0.0.1:3000;
    keepalive 64;
}

# Cache zone
proxy_cache_path /var/cache/nginx/nextjs levels=1:2 keys_zone=NEXTJS:100m inactive=7d use_temp_path=off;

server {
    listen 80;
    listen [::]:80;
    server_name example.com www.example.com;

    # Gzip
    gzip on;
    gzip_proxied any;
    gzip_comp_level 6;
    gzip_types text/plain text/css text/xml application/json application/javascript application/rss+xml application/atom+xml image/svg+xml;

    # Static files from Next.js
    location /_next/static {
        proxy_cache NEXTJS;
        proxy_pass http://nextjs_upstream;
        add_header X-Cache-Status $upstream_cache_status;
        expires 365d;
        access_log off;
    }

    # Static public files
    location /static {
        alias /var/www/nextjs-production/public/static;
        expires 365d;
        access_log off;
    }

    # Image optimization
    location /_next/image {
        proxy_cache NEXTJS;
        proxy_cache_valid 200 60m;
        proxy_pass http://nextjs_upstream;
    }

    # API routes - no cache
    location /api {
        proxy_pass http://nextjs_upstream;
        proxy_http_version 1.1;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }

    # Main application
    location / {
        proxy_pass http://nextjs_upstream;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_cache_bypass $http_upgrade;
    }
}
```

### Enable Site

```bash
sudo ln -s /etc/nginx/sites-available/nextjs-app /etc/nginx/sites-enabled/
sudo nginx -t
sudo systemctl reload nginx
```

---

## 6. SSL with Certbot

```bash
sudo certbot --nginx -d example.com -d www.example.com
```

---

## 7. Static Export Deployment

For SSG-only sites, serve directly with Nginx:

```nginx
server {
    listen 80;
    server_name example.com www.example.com;
    root /var/www/nextjs-app/out;
    index index.html;

    # Handle client-side routing
    location / {
        try_files $uri $uri.html $uri/ /index.html;
    }

    # Cache static assets
    location /_next/static {
        expires 365d;
        add_header Cache-Control "public, immutable";
    }

    location ~* \.(js|css|png|jpg|jpeg|gif|ico|svg|woff|woff2)$ {
        expires 30d;
        add_header Cache-Control "public";
    }

    # Security headers
    add_header X-Frame-Options "SAMEORIGIN" always;
    add_header X-Content-Type-Options "nosniff" always;
}
```

---

## 8. Deployment Script

```bash
nano /var/www/nextjs-app/deploy.sh
```

```bash
#!/bin/bash
set -e

APP_DIR="/var/www/nextjs-app"
PROD_DIR="/var/www/nextjs-production"
BRANCH="main"

echo "🚀 Starting Next.js deployment..."

cd $APP_DIR

echo "📥 Pulling latest changes..."
git fetch origin
git reset --hard origin/$BRANCH

echo "📦 Installing dependencies..."
npm ci

echo "🔨 Building application..."
npm run build

echo "📂 Copying standalone build..."
rm -rf $PROD_DIR/*
cp -r .next/standalone/* $PROD_DIR/
cp -r .next/static $PROD_DIR/.next/
cp -r public $PROD_DIR/

echo "🔄 Reloading PM2..."
pm2 reload nextjs-app

echo "✅ Deployment complete!"
```

```bash
chmod +x /var/www/nextjs-app/deploy.sh
```

---

## 9. Environment Variables in Production

### Runtime Environment Variables

```javascript
// For runtime (server-side) variables
// Access via process.env.VARIABLE_NAME

// next.config.js
module.exports = {
  env: {
    CUSTOM_VAR: process.env.CUSTOM_VAR,
  },
};
```

### Build-time Environment Variables

```bash
# Variables prefixed with NEXT_PUBLIC_ are available in browser
NEXT_PUBLIC_API_URL=https://api.example.com
```

### Using dotenv

```bash
# .env.local - local development
# .env.production - production build
# .env - default
```

---

## 10. Health Check Endpoint

Create `app/api/health/route.ts` (App Router):

```typescript
import { NextResponse } from 'next/server';

export async function GET() {
  return NextResponse.json({
    status: 'healthy',
    timestamp: new Date().toISOString(),
    version: process.env.npm_package_version || '1.0.0'
  });
}
```

Or `pages/api/health.ts` (Pages Router):

```typescript
import type { NextApiRequest, NextApiResponse } from 'next';

export default function handler(req: NextApiRequest, res: NextApiResponse) {
  res.status(200).json({ status: 'healthy' });
}
```

---

## 11. Troubleshooting

### Build Errors

```bash
# Clear cache and rebuild
rm -rf .next node_modules
npm install
npm run build
```

### Memory Issues

```bash
# Increase Node.js memory
NODE_OPTIONS="--max-old-space-size=4096" npm run build

# In ecosystem.config.js
node_args: '--max-old-space-size=4096'
```

### 500 Errors

```bash
# Check PM2 logs
pm2 logs nextjs-app

# Check Nginx error logs
sudo tail -f /var/log/nginx/error.log
```

### API Routes Not Working

```bash
# Ensure API routes are in correct directory
# App Router: app/api/
# Pages Router: pages/api/

# Check proxy pass in Nginx
location /api {
    proxy_pass http://localhost:3000;
}
```

---

## 12. Performance Optimization

### Image Optimization

```javascript
// next.config.js
module.exports = {
  images: {
    domains: ['example.com', 'cdn.example.com'],
    formats: ['image/avif', 'image/webp'],
    deviceSizes: [640, 750, 828, 1080, 1200, 1920, 2048, 3840],
  },
};
```

### Bundle Analysis

```bash
npm install @next/bundle-analyzer

# next.config.js
const withBundleAnalyzer = require('@next/bundle-analyzer')({
  enabled: process.env.ANALYZE === 'true',
});

module.exports = withBundleAnalyzer({
  // config
});

# Run analysis
ANALYZE=true npm run build
```

---

## Quick Reference

| Task | Command |
|------|---------|
| Build | `npm run build` |
| Start prod | `npm start` or `pm2 start` |
| View logs | `pm2 logs nextjs-app` |
| Restart | `pm2 restart nextjs-app` |
| Zero-downtime reload | `pm2 reload nextjs-app` |

---

## Next Steps

1. **[CI/CD Setup](./github-actions-deploy.md)** - Automated deployments
2. **[SSL Setup](./certbot-installation.md)** - HTTPS with Certbot
3. **[Monitoring](./server-monitoring-guide.md)** - Application monitoring

---

✅ Your Next.js application is deployed!
