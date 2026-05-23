# VPS Server Setup Guides

Simple, step-by-step guides for setting up and deploying apps on a VPS or AWS EC2 server.

---

## Start Here (Do These in Order)

| Step | Guide | What It Does |
|------|-------|-------------|
| 1 | [Initial Server Setup](./01-initial-server-setup.md) | Connect, create user, secure SSH, add swap |
| 2 | [Firewall Setup](./02-firewall-configuration.md) | Block unwanted traffic |
| 3 | [SSH Keys & Fail2Ban](./03-ssh-keys-and-security.md) | Key login + block brute force attacks |
| 4 | [Install Nginx](./nginx-complete-guide.md) | Web server setup |
| 5 | [SSL / HTTPS](./certbot-installation.md) | Free SSL certificate |

---

## Deploy Your App

| App | Guide |
|-----|-------|
| Node.js / Express | [deploy-nodejs-app.md](./deploy-nodejs-app.md) |
| Next.js (SSR) | [deploy-nextjs-app.md](./deploy-nextjs-app.md) |
| Python Flask/Django | [deploy-python-flask-django.md](./deploy-python-flask-django.md) |
| PHP / Laravel | [deploy-php-laravel.md](./deploy-php-laravel.md) |
| Static Sites (React, Vue) | [deploy-static-sites.md](./deploy-static-sites.md) |

---

## Databases

| Database | Guide |
|----------|-------|
| MongoDB | [install-mongodb.md](./install-mongodb.md) |
| MySQL / MariaDB | [install-mysql-mariadb.md](./install-mysql-mariadb.md) |
| PostgreSQL | [install-postgresql.md](./install-postgresql.md) |
| Redis | [install-redis.md](./install-redis.md) |

---

## Tools & Automation

| Tool | Guide |
|------|-------|
| Docker | [docker-complete-guide.md](./docker-complete-guide.md) |
| PM2 (process manager) | [pm2-advanced-guide.md](./pm2-advanced-guide.md) |
| GitHub Actions (auto-deploy) | [github-actions-deploy.md](./github-actions-deploy.md) |
| Environment Variables | [environment-variables-guide.md](./environment-variables-guide.md) |

---

## Monitoring & Backups

| Guide | What It Covers |
|-------|---------------|
| [Server Monitoring](./server-monitoring-guide.md) | CPU, memory, disk alerts |
| [Backup Strategies](./backup-strategies.md) | Automated backups |
| [Security Hardening](./server-security-hardening.md) | Extra security steps |

---

## AWS EC2

[AWS EC2 Launch Guide](./aws-ec2-launch-guide.md) — Create and set up an EC2 instance from scratch.

---

**Last Updated:** May 2026
