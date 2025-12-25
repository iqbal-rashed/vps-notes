# GitHub Actions Deploy

Automated deployment with GitHub Actions for VPS/EC2.

---

## 1. Basic SSH Deploy

Create `.github/workflows/deploy.yml`:

```yaml
name: Deploy to VPS

on:
  push:
    branches: [main]

jobs:
  deploy:
    runs-on: ubuntu-latest
    
    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Deploy via SSH
        uses: appleboy/ssh-action@v1.0.3
        with:
          host: ${{ secrets.HOST }}
          username: ${{ secrets.USERNAME }}
          key: ${{ secrets.SSH_PRIVATE_KEY }}
          port: 22
          script: |
            cd /var/www/myapp
            git pull origin main
            npm ci
            npm run build
            pm2 reload myapp
```

---

## 2. GitHub Secrets

Add secrets in **Settings > Secrets > Actions**:

| Secret | Value |
|--------|-------|
| `HOST` | Your server IP |
| `USERNAME` | SSH username (e.g., deployer) |
| `SSH_PRIVATE_KEY` | Your private key content |

Generate deploy key:
```bash
ssh-keygen -t ed25519 -C "github-actions" -f ~/.ssh/deploy_key
cat ~/.ssh/deploy_key      # Add to GitHub secrets
cat ~/.ssh/deploy_key.pub  # Add to server ~/.ssh/authorized_keys
```

---

## 3. Node.js with Build

```yaml
name: Build and Deploy

on:
  push:
    branches: [main]

jobs:
  build:
    runs-on: ubuntu-latest
    
    steps:
      - uses: actions/checkout@v4
      
      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'npm'
      
      - name: Install dependencies
        run: npm ci
      
      - name: Run tests
        run: npm test
      
      - name: Build
        run: npm run build
      
      - name: Deploy
        uses: appleboy/ssh-action@v1.0.3
        with:
          host: ${{ secrets.HOST }}
          username: ${{ secrets.USERNAME }}
          key: ${{ secrets.SSH_PRIVATE_KEY }}
          script: |
            cd /var/www/myapp
            git pull
            npm ci --production
            npm run build
            pm2 reload myapp
```

---

## 4. Docker Deploy

```yaml
name: Docker Deploy

on:
  push:
    branches: [main]

jobs:
  build-and-deploy:
    runs-on: ubuntu-latest
    
    steps:
      - uses: actions/checkout@v4
      
      - name: Login to Docker Hub
        uses: docker/login-action@v3
        with:
          username: ${{ secrets.DOCKER_USERNAME }}
          password: ${{ secrets.DOCKER_TOKEN }}
      
      - name: Build and push
        uses: docker/build-push-action@v5
        with:
          push: true
          tags: username/myapp:latest
      
      - name: Deploy
        uses: appleboy/ssh-action@v1.0.3
        with:
          host: ${{ secrets.HOST }}
          username: ${{ secrets.USERNAME }}
          key: ${{ secrets.SSH_PRIVATE_KEY }}
          script: |
            docker pull username/myapp:latest
            docker stop myapp || true
            docker rm myapp || true
            docker run -d --name myapp -p 3000:3000 username/myapp:latest
```

---

## 5. Deploy with rsync

```yaml
name: Rsync Deploy

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
      
      - run: npm ci
      - run: npm run build
      
      - name: Deploy with rsync
        uses: burnett01/rsync-deployments@6.0.0
        with:
          switches: -avzr --delete
          path: dist/
          remote_path: /var/www/myapp/
          remote_host: ${{ secrets.HOST }}
          remote_user: ${{ secrets.USERNAME }}
          remote_key: ${{ secrets.SSH_PRIVATE_KEY }}
      
      - name: Restart app
        uses: appleboy/ssh-action@v1.0.3
        with:
          host: ${{ secrets.HOST }}
          username: ${{ secrets.USERNAME }}
          key: ${{ secrets.SSH_PRIVATE_KEY }}
          script: pm2 reload myapp
```

---

## 6. Environment Variables

```yaml
jobs:
  deploy:
    runs-on: ubuntu-latest
    env:
      NODE_ENV: production
    
    steps:
      - uses: actions/checkout@v4
      
      - name: Create .env file
        run: |
          echo "DATABASE_URL=${{ secrets.DATABASE_URL }}" >> .env
          echo "API_KEY=${{ secrets.API_KEY }}" >> .env
```

---

## 7. Conditional Deploy

```yaml
on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - run: npm ci
      - run: npm test
  
  deploy:
    needs: test
    if: github.ref == 'refs/heads/main' && github.event_name == 'push'
    runs-on: ubuntu-latest
    steps:
      - name: Deploy
        # deploy steps...
```

---

## Quick Reference

```yaml
# Useful actions
actions/checkout@v4          # Checkout code
actions/setup-node@v4        # Setup Node.js
appleboy/ssh-action@v1.0.3   # SSH commands
docker/build-push-action@v5  # Docker build & push
```

---

✅ CI/CD is ready!
