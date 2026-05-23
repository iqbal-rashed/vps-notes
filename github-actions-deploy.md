# Auto-Deploy with GitHub Actions

Set up automatic deployment — every time you push to the `main` branch, your server automatically pulls and deploys the latest code.

---

## How It Works

1. You push code to GitHub
2. GitHub Actions runs your workflow
3. It SSHs into your server and runs deploy commands
4. Your app reloads with zero downtime

---

## Step 1 — Create a Deploy SSH Key

Run this on your **server** (not your local machine):

```bash
# Create a key specifically for GitHub Actions
ssh-keygen -t ed25519 -C "github-actions" -f ~/.ssh/deploy_key -N ""

# Add the public key to authorized_keys so GitHub can log in
cat ~/.ssh/deploy_key.pub >> ~/.ssh/authorized_keys

# Show the private key — you'll paste this into GitHub
cat ~/.ssh/deploy_key
```

---

## Step 2 — Add Secrets to GitHub

Go to your GitHub repo → **Settings** → **Secrets and variables** → **Actions** → **New repository secret**

Add these three secrets:

| Secret Name | Value |
|-------------|-------|
| `HOST` | Your server IP address |
| `USERNAME` | Your SSH username (e.g., `deployer`) |
| `SSH_PRIVATE_KEY` | The private key content from `cat ~/.ssh/deploy_key` |

If you changed your SSH port to 2222, also add:

| Secret Name | Value |
|-------------|-------|
| `PORT` | `2222` |

---

## Step 3 — Create the Workflow File

In your project, create this file: `.github/workflows/deploy.yml`

### For a Node.js / Express App

```yaml
name: Deploy to VPS

on:
  push:
    branches: [main]

jobs:
  deploy:
    runs-on: ubuntu-latest

    steps:
      - name: Deploy via SSH
        uses: appleboy/ssh-action@v1.0.3
        with:
          host: ${{ secrets.HOST }}
          username: ${{ secrets.USERNAME }}
          key: ${{ secrets.SSH_PRIVATE_KEY }}
          port: ${{ secrets.PORT || 22 }}
          script: |
            cd /var/www/myapp
            git pull origin main
            npm ci --production
            pm2 reload myapp
```

### For a Next.js App (With Build Step)

```yaml
name: Deploy Next.js

on:
  push:
    branches: [main]

jobs:
  deploy:
    runs-on: ubuntu-latest

    steps:
      - name: Deploy via SSH
        uses: appleboy/ssh-action@v1.0.3
        with:
          host: ${{ secrets.HOST }}
          username: ${{ secrets.USERNAME }}
          key: ${{ secrets.SSH_PRIVATE_KEY }}
          port: ${{ secrets.PORT || 22 }}
          script: |
            cd /var/www/nextjs-app
            git pull origin main
            npm ci
            npm run build
            cp -r .next/standalone/. /var/www/nextjs-production/
            cp -r .next/static /var/www/nextjs-production/.next/
            cp -r public /var/www/nextjs-production/
            pm2 reload nextjs-app
```

### Run Tests Before Deploying

```yaml
name: Test and Deploy

on:
  push:
    branches: [main]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'npm'

      - name: Install and Test
        run: |
          npm ci
          npm test

  deploy:
    needs: test          # Only deploy if tests pass
    runs-on: ubuntu-latest
    steps:
      - name: Deploy via SSH
        uses: appleboy/ssh-action@v1.0.3
        with:
          host: ${{ secrets.HOST }}
          username: ${{ secrets.USERNAME }}
          key: ${{ secrets.SSH_PRIVATE_KEY }}
          port: ${{ secrets.PORT || 22 }}
          script: |
            cd /var/www/myapp
            git pull origin main
            npm ci --production
            pm2 reload myapp
```

---

## Watch It Run

After pushing to `main`, go to your repo → **Actions** tab to see the workflow running. Green check = deployed. Red X = something failed.

Click the run to see the logs and find errors.

---

## Troubleshooting

**"Permission denied" when SSH connects:**
- Check that `deploy_key.pub` is in the server's `~/.ssh/authorized_keys`
- Check `SSH_PRIVATE_KEY` secret has the full private key including the header/footer lines
- Verify the `USERNAME` secret matches your server user

**"Host key verification failed":**
- Add the server to known hosts. Run on your server:
```bash
ssh-keyscan -p 2222 YOUR_SERVER_IP
```
Add the output as a `KNOWN_HOSTS` secret, then add this step to your workflow before the deploy step:
```yaml
- name: Add server to known hosts
  run: |
    mkdir -p ~/.ssh
    echo "${{ secrets.KNOWN_HOSTS }}" >> ~/.ssh/known_hosts
```

**Deployment succeeds but app not updating:**
```bash
# SSH to server and check PM2
pm2 logs myapp --lines 50
```

---

## Quick Reference

| File | Location in Repo |
|------|-----------------|
| Workflow file | `.github/workflows/deploy.yml` |
| GitHub Secrets | Repo → Settings → Secrets → Actions |
| Workflow runs | Repo → Actions tab |
