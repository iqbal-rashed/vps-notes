# Install Redis

Redis is an in-memory data store. Apps use it for caching (storing temporary data in RAM for fast access), sessions, and queues.

---

## Step 1 — Install

```bash
sudo apt update
sudo apt install redis-server -y
```

---

## Step 2 — Configure Redis

```bash
sudo nano /etc/redis/redis.conf
```

Find and update these settings:

```ini
# Only listen on localhost (not accessible from outside)
bind 127.0.0.1 ::1

# Set a password (highly recommended)
requirepass YourStrongRedisPassword!

# Limit memory usage (adjust to your server size)
maxmemory 256mb

# When memory is full, remove old/least-used data
maxmemory-policy allkeys-lru

# Save data to disk (so it survives restarts)
appendonly yes
appendfsync everysec

# Disable dangerous commands that could wipe all your data
rename-command FLUSHDB ""
rename-command FLUSHALL ""
rename-command CONFIG ""
```

---

## Step 3 — Start Redis

```bash
sudo systemctl restart redis-server
sudo systemctl enable redis-server
sudo systemctl status redis-server
```

---

## Step 4 — Test It Works

```bash
redis-cli -a YourStrongRedisPassword! ping
# Should print: PONG
```

**Connection string for your app's `.env`:**
```
REDIS_URL=redis://:YourStrongRedisPassword!@localhost:6379
```

---

## Basic Redis Commands

```bash
redis-cli -a YOUR_PASSWORD

# Set a value
SET mykey "hello"

# Get a value
GET mykey

# Set with expiration (3600 seconds = 1 hour)
SETEX sessionkey 3600 "user_data"

# Check time left before it expires
TTL sessionkey

# Delete a key
DEL mykey

# See all keys (don't use in production on large datasets)
KEYS *

# Check memory usage
INFO memory
```

---

## Using Redis in Node.js

```javascript
const redis = require('redis');

const client = redis.createClient({
  url: process.env.REDIS_URL
});

await client.connect();

// Cache something for 1 hour
await client.setEx('user:123', 3600, JSON.stringify(userData));

// Get it back
const cached = await client.get('user:123');
```

---

## Backup

```bash
# Trigger a manual save
redis-cli -a YOUR_PASSWORD BGSAVE

# The backup file is at:
ls -lh /var/lib/redis/dump.rdb

# Copy it somewhere safe
cp /var/lib/redis/dump.rdb /backup/redis_$(date +%Y%m%d).rdb
```

---

## Troubleshooting

**Redis not starting:**
```bash
sudo tail -20 /var/log/redis/redis-server.log
```

**Connection refused:**
```bash
sudo systemctl status redis-server
sudo ss -tlnp | grep 6379
```

**Authentication error:**
Check your password in `/etc/redis/redis.conf` under `requirepass`.

---

## Quick Reference

| Task | Command |
|------|---------|
| Start | `sudo systemctl start redis-server` |
| Status | `sudo systemctl status redis-server` |
| Connect | `redis-cli -a YOUR_PASSWORD` |
| Ping test | `redis-cli -a YOUR_PASSWORD ping` |
| Config | `/etc/redis/redis.conf` |
