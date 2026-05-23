# Environment Variables

Environment variables store secrets and settings your app needs — database passwords, API keys, etc. They're kept separate from your code so you don't accidentally commit them to GitHub.

---

## The .env File

Create a `.env` file in your app folder:

```bash
nano /var/www/myapp/.env
```

```env
NODE_ENV=production
PORT=3000
DATABASE_URL=mongodb://appuser:password@localhost:27017/myapp
JWT_SECRET=a-very-long-random-secret-string
API_KEY=your-api-key-here
```

**Important — secure it immediately:**
```bash
chmod 600 /var/www/myapp/.env
```

This makes it only readable by you, not other users on the server.

---

## Make Sure .env Is Not in Git

Add it to `.gitignore` in your project:

```bash
echo ".env" >> .gitignore
echo ".env.production" >> .gitignore
echo ".env.local" >> .gitignore
```

> Never commit `.env` files to GitHub. Secrets in git history can be found even after deletion.

---

## Using .env in Node.js

Install the dotenv package:
```bash
npm install dotenv
```

At the very top of your main file (`app.js` or `index.js`):
```javascript
require('dotenv').config();

// Now use your variables
const dbUrl = process.env.DATABASE_URL;
const port = process.env.PORT || 3000;
```

---

## Using .env with PM2

PM2 can load your `.env` file automatically. In `ecosystem.config.js`:

```javascript
module.exports = {
  apps: [{
    name: 'myapp',
    script: './app.js',
    env_file: '/var/www/myapp/.env',
    env: {
      NODE_ENV: 'production'
    }
  }]
};
```

Or pass the env file when starting:
```bash
pm2 start app.js --name myapp --env-file .env
```

---

## Next.js Environment Variables

Next.js has a specific naming rule:

```env
# Only available on the server (keep secrets here)
DATABASE_URL=mongodb://...
JWT_SECRET=secret

# Available in the browser too (safe for public data only)
NEXT_PUBLIC_API_URL=https://api.example.com
NEXT_PUBLIC_SITE_NAME=My App
```

> Anything starting with `NEXT_PUBLIC_` is visible to all users. Never put secrets there.

---

## Updating Environment Variables

1. Edit the `.env` file on the server
2. Reload your app to pick up changes

```bash
nano /var/www/myapp/.env
# Make your changes, save

pm2 reload myapp
```

---

## Checking Current Environment Variables

```bash
# Show all environment variables (careful — shows secrets)
printenv

# Check a specific variable
printenv DATABASE_URL

# In Node.js
node -e "require('dotenv').config(); console.log(process.env.DATABASE_URL)"
```

---

## Security Tips

- Never hard-code secrets in your code files
- Never commit `.env` to GitHub
- Use `chmod 600 .env` to restrict access
- Use different secrets for development and production
- Generate secure random secrets with:
```bash
openssl rand -hex 32
```

---

## Quick Reference

| File | Purpose |
|------|---------|
| `.env` | Default (used if nothing else matches) |
| `.env.production` | Production settings |
| `.env.development` | Development settings |
| `.env.local` | Local overrides (never committed) |
