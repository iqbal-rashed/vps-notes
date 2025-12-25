# Install Redis

Complete guide for installing and configuring Redis on Ubuntu/Debian.

---

## 1. Install Redis

```bash
sudo apt update
sudo apt install redis-server -y
```

---

## 2. Configure Redis

```bash
sudo nano /etc/redis/redis.conf
```

Key settings:

```ini
# Bind to localhost only
bind 127.0.0.1 ::1

# Set password (recommended)
requirepass YourStrongPassword123!

# Memory limit
maxmemory 256mb
maxmemory-policy allkeys-lru

# Persistence
appendonly yes
appendfsync everysec

# Disable dangerous commands
rename-command FLUSHDB ""
rename-command FLUSHALL ""
rename-command DEBUG ""
```

---

## 3. Start Redis

```bash
sudo systemctl restart redis-server
sudo systemctl enable redis-server
sudo systemctl status redis-server
```

---

## 4. Connect to Redis

```bash
# Without password
redis-cli

# With password
redis-cli -a YourStrongPassword123!

# Test
redis-cli ping
# Should return: PONG
```

---

## 5. Basic Commands

```bash
# Set/Get
SET mykey "Hello"
GET mykey

# Expiration
SETEX mykey 3600 "value"  # Expires in 1 hour
TTL mykey

# Delete
DEL mykey

# List all keys
KEYS *

# Database info
INFO
INFO memory
```

---

## 6. Connection String

```
redis://:password@localhost:6379/0
```

Node.js example:
```javascript
const redis = require('redis');
const client = redis.createClient({
  url: 'redis://:password@localhost:6379'
});
```

---

## 7. Remote Access (If Needed)

```bash
# In redis.conf
bind 0.0.0.0
requirepass YourStrongPassword123!

# Firewall
sudo ufw allow from 10.0.0.5 to any port 6379

sudo systemctl restart redis-server
```

---

## 8. Backup

```bash
# Create backup
redis-cli BGSAVE

# Backup file location
ls -la /var/lib/redis/dump.rdb

# Copy backup
cp /var/lib/redis/dump.rdb /backup/redis_$(date +%Y%m%d).rdb
```

---

## Quick Reference

| Task | Command |
|------|---------|
| Start | `sudo systemctl start redis-server` |
| Status | `sudo systemctl status redis-server` |
| Connect | `redis-cli` |
| Ping | `redis-cli ping` |
| Config | `/etc/redis/redis.conf` |

---

✅ Redis is ready!
