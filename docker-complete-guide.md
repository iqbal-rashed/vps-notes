# Docker Complete Guide

Complete guide for Docker installation, images, containers, and Docker Compose.

---

## 1. Install Docker

```bash
# Remove old versions
sudo apt remove docker docker-engine docker.io containerd runc

# Install prerequisites
sudo apt update
sudo apt install ca-certificates curl gnupg -y

# Add Docker GPG key
sudo install -m 0755 -d /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
sudo chmod a+r /etc/apt/keyrings/docker.gpg

# Add repository
echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

# Install Docker
sudo apt update
sudo apt install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin -y

# Verify
docker --version
docker compose version
```

---

## 2. Post-Installation

```bash
# Add user to docker group (no sudo needed)
sudo usermod -aG docker $USER

# Apply changes (logout/login or run)
newgrp docker

# Test
docker run hello-world
```

---

## 3. Docker Commands

### Container Management

```bash
# Run container
docker run -d --name myapp -p 3000:3000 node:20

# Run with environment variables
docker run -d --name myapp -p 3000:3000 -e NODE_ENV=production myimage

# List containers
docker ps        # Running
docker ps -a     # All

# Stop/Start/Restart
docker stop myapp
docker start myapp
docker restart myapp

# Remove container
docker rm myapp
docker rm -f myapp  # Force

# View logs
docker logs myapp
docker logs -f myapp  # Follow

# Execute command in container
docker exec -it myapp bash
docker exec -it myapp sh
```

### Image Management

```bash
# List images
docker images

# Pull image
docker pull node:20

# Build image
docker build -t myapp:latest .

# Remove image
docker rmi myapp:latest

# Remove unused images
docker image prune
docker image prune -a  # All unused
```

### Volumes

```bash
# Create volume
docker volume create mydata

# List volumes
docker volume ls

# Run with volume
docker run -d -v mydata:/app/data myapp

# Bind mount
docker run -d -v $(pwd)/data:/app/data myapp
```

### Networks

```bash
# Create network
docker network create mynetwork

# Run with network
docker run -d --network mynetwork --name myapp myimage

# List networks
docker network ls
```

---

## 4. Dockerfile

### Node.js Application

```dockerfile
# Base image
FROM node:20-alpine

# Set working directory
WORKDIR /app

# Copy package files
COPY package*.json ./

# Install dependencies
RUN npm ci --only=production

# Copy source code
COPY . .

# Build (if needed)
RUN npm run build

# Expose port
EXPOSE 3000

# Start command
CMD ["node", "dist/index.js"]
```

### Multi-Stage Build

```dockerfile
# Build stage
FROM node:20-alpine AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build

# Production stage
FROM node:20-alpine
WORKDIR /app
COPY --from=builder /app/dist ./dist
COPY --from=builder /app/node_modules ./node_modules
COPY package*.json ./
EXPOSE 3000
CMD ["node", "dist/index.js"]
```

### .dockerignore

```
node_modules
npm-debug.log
.git
.gitignore
.env
Dockerfile
.dockerignore
README.md
```

---

## 5. Docker Compose

### Basic docker-compose.yml

```yaml
version: '3.8'

services:
  app:
    build: .
    ports:
      - "3000:3000"
    environment:
      - NODE_ENV=production
      - DATABASE_URL=mongodb://mongo:27017/myapp
    depends_on:
      - mongo
    restart: unless-stopped

  mongo:
    image: mongo:7
    volumes:
      - mongo_data:/data/db
    restart: unless-stopped

volumes:
  mongo_data:
```

### Full Stack Example

```yaml
version: '3.8'

services:
  nginx:
    image: nginx:alpine
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - ./nginx.conf:/etc/nginx/nginx.conf:ro
      - ./ssl:/etc/nginx/ssl:ro
    depends_on:
      - app
    restart: unless-stopped

  app:
    build: .
    expose:
      - "3000"
    environment:
      - NODE_ENV=production
      - DATABASE_URL=postgresql://user:pass@db:5432/myapp
      - REDIS_URL=redis://redis:6379
    depends_on:
      - db
      - redis
    restart: unless-stopped

  db:
    image: postgres:16
    environment:
      - POSTGRES_USER=user
      - POSTGRES_PASSWORD=pass
      - POSTGRES_DB=myapp
    volumes:
      - postgres_data:/var/lib/postgresql/data
    restart: unless-stopped

  redis:
    image: redis:alpine
    command: redis-server --appendonly yes
    volumes:
      - redis_data:/data
    restart: unless-stopped

volumes:
  postgres_data:
  redis_data:
```

### Docker Compose Commands

```bash
# Start services
docker compose up -d

# Stop services
docker compose down

# View logs
docker compose logs -f

# Rebuild
docker compose up -d --build

# Scale service
docker compose up -d --scale app=3

# Execute command
docker compose exec app bash
```

---

## 6. Production Tips

### Health Checks

```yaml
services:
  app:
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:3000/health"]
      interval: 30s
      timeout: 10s
      retries: 3
      start_period: 40s
```

### Resource Limits

```yaml
services:
  app:
    deploy:
      resources:
        limits:
          cpus: '0.5'
          memory: 512M
        reservations:
          memory: 256M
```

### Restart Policies

```yaml
services:
  app:
    restart: unless-stopped  # or always, on-failure
```

---

## 7. Cleanup

```bash
# Remove stopped containers
docker container prune

# Remove unused images
docker image prune -a

# Remove unused volumes
docker volume prune

# Remove everything unused
docker system prune -a --volumes
```

---

## Quick Reference

| Task | Command |
|------|---------|
| Run container | `docker run -d -p 3000:3000 image` |
| List containers | `docker ps` |
| View logs | `docker logs -f container` |
| Execute shell | `docker exec -it container bash` |
| Build image | `docker build -t name .` |
| Compose up | `docker compose up -d` |
| Compose down | `docker compose down` |

---

✅ Docker is ready!
