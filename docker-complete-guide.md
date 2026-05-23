# Docker Guide

Docker lets you package your app and all its dependencies into a "container" — it runs the same way everywhere.

**When to use Docker:** When you want your dev environment to match production exactly, or when deploying multiple apps on one server.

---

## Install Docker

```bash
# Remove old versions if any
sudo apt remove docker docker.io containerd runc 2>/dev/null

# Install prerequisites
sudo apt update
sudo apt install ca-certificates curl gnupg -y

# Add Docker's official key and repo
sudo install -m 0755 -d /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
sudo chmod a+r /etc/apt/keyrings/docker.gpg

echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

# Install Docker
sudo apt update
sudo apt install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin -y

# Check it works
docker --version
docker compose version
```

**Run Docker without sudo:**
```bash
sudo usermod -aG docker $USER
newgrp docker   # Apply immediately without logging out
```

---

## Basic Docker Commands

```bash
# Run a container
docker run -d --name myapp -p 3000:3000 myimage

# See running containers
docker ps

# See all containers (including stopped)
docker ps -a

# Stop / Start / Remove
docker stop myapp
docker start myapp
docker rm myapp

# View logs
docker logs myapp
docker logs -f myapp    # Follow live

# Open a shell inside a container
docker exec -it myapp bash

# See images you've downloaded/built
docker images

# Clean up unused images and containers (free disk space)
docker system prune
```

---

## Write a Dockerfile

A `Dockerfile` is a recipe for building your app's image.

### Node.js App

```dockerfile
FROM node:20-alpine

WORKDIR /app

# Install dependencies first (cached unless package.json changes)
COPY package*.json ./
RUN npm ci --only=production

# Copy source code
COPY . .

EXPOSE 3000
CMD ["node", "app.js"]
```

### Next.js App

```dockerfile
FROM node:20-alpine AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build

FROM node:20-alpine
WORKDIR /app
COPY --from=builder /app/.next/standalone ./
COPY --from=builder /app/.next/static ./.next/static
COPY --from=builder /app/public ./public
EXPOSE 3000
CMD ["node", "server.js"]
```

### .dockerignore (Important!)

Create this file so Docker doesn't copy unnecessary files:

```
node_modules
.git
.env
*.log
.next
dist
```

Build and run:
```bash
docker build -t myapp .
docker run -d --name myapp -p 3000:3000 myapp
```

---

## Docker Compose (Multiple Services Together)

Docker Compose lets you run your app + database + cache together with one command.

Create `docker-compose.yml`:

```yaml
services:
  app:
    build: .
    ports:
      - "3000:3000"
    env_file:
      - .env
    depends_on:
      - mongo
    restart: unless-stopped

  mongo:
    image: mongo:7
    volumes:
      - mongo_data:/data/db
    environment:
      MONGO_INITDB_ROOT_USERNAME: admin
      MONGO_INITDB_ROOT_PASSWORD: StrongPassword!
    restart: unless-stopped

volumes:
  mongo_data:
```

Commands:
```bash
# Start all services
docker compose up -d

# Stop all services
docker compose down

# View logs
docker compose logs -f

# Rebuild after code changes
docker compose up -d --build

# See status
docker compose ps
```

---

## Nginx + App with Docker Compose

Full example with Nginx in front:

```yaml
services:
  nginx:
    image: nginx:alpine
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - ./nginx.conf:/etc/nginx/conf.d/default.conf:ro
    depends_on:
      - app
    restart: unless-stopped

  app:
    build: .
    expose:
      - "3000"    # Only expose inside the Docker network, not to the internet
    env_file:
      - .env
    restart: unless-stopped

  db:
    image: postgres:16-alpine
    environment:
      POSTGRES_USER: appuser
      POSTGRES_PASSWORD: StrongPassword!
      POSTGRES_DB: myapp
    volumes:
      - postgres_data:/var/lib/postgresql/data
    restart: unless-stopped

volumes:
  postgres_data:
```

---

## Security Tips for Docker

- Never run containers as root — add `USER node` or `USER appuser` in your Dockerfile
- Never put secrets in your `Dockerfile` — use `.env` files or Docker secrets
- Keep the `.env` file in `.dockerignore` so it doesn't get baked into the image
- Databases in Docker should not have ports published (`ports:`) — only `expose:` so they're only reachable within the container network
- Regularly update your base images: `docker pull node:20-alpine`

---

## Troubleshooting

**Container won't start:**
```bash
docker logs myapp
```

**Can't connect to app:**
```bash
# Check the port mapping
docker ps
# Make sure ports shows: 0.0.0.0:3000->3000/tcp
```

**Out of disk space:**
```bash
docker system prune -a   # Removes all unused images, containers, networks
```

**Rebuild after code change:**
```bash
docker compose up -d --build
```

---

## Quick Reference

| Task | Command |
|------|---------|
| Start all (Compose) | `docker compose up -d` |
| Stop all | `docker compose down` |
| Rebuild | `docker compose up -d --build` |
| Logs | `docker compose logs -f` |
| Shell into container | `docker exec -it myapp bash` |
| Free disk space | `docker system prune` |
