# Install MongoDB Community Edition

Complete guide for installing, securing, and managing MongoDB on Ubuntu/Debian servers.

---

## Prerequisites

- Ubuntu 20.04/22.04/24.04 LTS or Debian
- sudo/root access
- At least 2GB RAM (4GB recommended)

---

## 1. Install Prerequisites

```bash
sudo apt update
sudo apt install gnupg curl wget -y
```

---

## 2. Import MongoDB GPG Key

```bash
# MongoDB 8.0 (Latest)
curl -fsSL https://www.mongodb.org/static/pgp/server-8.0.asc | \
   sudo gpg -o /usr/share/keyrings/mongodb-server-8.0.gpg --dearmor
```

For older versions:
```bash
# MongoDB 7.0
curl -fsSL https://www.mongodb.org/static/pgp/server-7.0.asc | \
   sudo gpg -o /usr/share/keyrings/mongodb-server-7.0.gpg --dearmor
```

---

## 3. Add MongoDB Repository

### Ubuntu 24.04 (Noble)

```bash
echo "deb [ arch=amd64,arm64 signed-by=/usr/share/keyrings/mongodb-server-8.0.gpg ] https://repo.mongodb.org/apt/ubuntu noble/mongodb-org/8.0 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-8.0.list
```

### Ubuntu 22.04 (Jammy)

```bash
echo "deb [ arch=amd64,arm64 signed-by=/usr/share/keyrings/mongodb-server-8.0.gpg ] https://repo.mongodb.org/apt/ubuntu jammy/mongodb-org/8.0 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-8.0.list
```

### Ubuntu 20.04 (Focal)

```bash
echo "deb [ arch=amd64,arm64 signed-by=/usr/share/keyrings/mongodb-server-8.0.gpg ] https://repo.mongodb.org/apt/ubuntu focal/mongodb-org/8.0 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-8.0.list
```

---

## 4. Install MongoDB

```bash
sudo apt update
sudo apt install mongodb-org -y
```

### Verify Installation

```bash
mongod --version
mongosh --version
```

---

## 5. Start and Enable MongoDB

```bash
# Start MongoDB
sudo systemctl start mongod

# Enable on boot
sudo systemctl enable mongod

# Check status
sudo systemctl status mongod
```

---

## 6. Connect to MongoDB

```bash
# Using mongosh (new shell)
mongosh

# Connect to specific database
mongosh "mongodb://localhost:27017/mydb"
```

### Basic Shell Commands

```javascript
// Show databases
show dbs

// Use/create database
use mydb

// Show collections
show collections

// Insert document
db.users.insertOne({ name: "John", email: "john@example.com" })

// Find documents
db.users.find()
db.users.find({ name: "John" })

// Exit
exit
```

---

## 7. Enable Authentication (CRITICAL for Production)

### Create Admin User

```bash
mongosh
```

```javascript
use admin

db.createUser({
  user: "admin",
  pwd: passwordPrompt(),  // Prompts for password
  roles: [
    { role: "userAdminAnyDatabase", db: "admin" },
    { role: "readWriteAnyDatabase", db: "admin" },
    { role: "dbAdminAnyDatabase", db: "admin" },
    { role: "clusterAdmin", db: "admin" }
  ]
})
```

### Create Application User

```javascript
use myapp

db.createUser({
  user: "appuser",
  pwd: passwordPrompt(),
  roles: [
    { role: "readWrite", db: "myapp" }
  ]
})
```

### Enable Authentication

```bash
sudo nano /etc/mongod.conf
```

```yaml
security:
  authorization: enabled
```

### Restart MongoDB

```bash
sudo systemctl restart mongod
```

### Connect with Authentication

```bash
mongosh -u admin -p --authenticationDatabase admin

# Or with connection string
mongosh "mongodb://admin:password@localhost:27017/mydb?authSource=admin"
```

---

## 8. Configure MongoDB

### Main Configuration File

```bash
sudo nano /etc/mongod.conf
```

### Recommended Production Configuration

```yaml
# mongod.conf

# Storage
storage:
  dbPath: /var/lib/mongodb
  journal:
    enabled: true
  wiredTiger:
    engineConfig:
      cacheSizeGB: 1  # Adjust based on RAM

# Logging
systemLog:
  destination: file
  logAppend: true
  path: /var/log/mongodb/mongod.log
  logRotate: reopen

# Network
net:
  port: 27017
  bindIp: 127.0.0.1  # Only localhost
  # bindIp: 0.0.0.0  # All interfaces (requires auth!)
  maxIncomingConnections: 1000

# Security
security:
  authorization: enabled

# Operation Profiling
operationProfiling:
  mode: slowOp
  slowOpThresholdMs: 100

# Replication (optional)
# replication:
#   replSetName: "rs0"
```

---

## 9. Allow Remote Connections (If Needed)

> ⚠️ **Warning:** Only do this if authentication is enabled!

### Update Configuration

```bash
sudo nano /etc/mongod.conf
```

```yaml
net:
  port: 27017
  bindIp: 0.0.0.0  # Or specific IP
```

### Update Firewall

```bash
# Only allow from specific IP
sudo ufw allow from 10.0.0.5 to any port 27017

# Or for all (NOT recommended)
# sudo ufw allow 27017
```

### Restart MongoDB

```bash
sudo systemctl restart mongod
```

### Connection String for Remote

```
mongodb://appuser:password@server-ip:27017/myapp?authSource=myapp
```

---

## 10. Backup and Restore

### Create Backup

```bash
# Full backup
mongodump --uri="mongodb://admin:password@localhost:27017" --out=/backup/mongodb/$(date +%Y%m%d)

# Specific database
mongodump --uri="mongodb://admin:password@localhost:27017/mydb" --out=/backup/mongodb/

# Compressed backup
mongodump --uri="mongodb://admin:password@localhost:27017" --gzip --archive=/backup/mongodb/backup.gz
```

### Restore Backup

```bash
# Full restore
mongorestore --uri="mongodb://admin:password@localhost:27017" /backup/mongodb/20241225/

# Specific database
mongorestore --uri="mongodb://admin:password@localhost:27017" --db mydb /backup/mongodb/mydb/

# From compressed archive
mongorestore --uri="mongodb://admin:password@localhost:27017" --gzip --archive=/backup/mongodb/backup.gz
```

### Automated Backup Script

```bash
#!/bin/bash
# /usr/local/bin/mongodb-backup.sh

BACKUP_DIR="/backup/mongodb"
MONGO_URI="mongodb://admin:password@localhost:27017"
DATE=$(date +%Y%m%d_%H%M%S)
RETENTION_DAYS=7

# Create backup
mongodump --uri="$MONGO_URI" --gzip --archive="$BACKUP_DIR/backup_$DATE.gz"

# Remove old backups
find $BACKUP_DIR -name "backup_*.gz" -mtime +$RETENTION_DAYS -delete

echo "Backup completed: backup_$DATE.gz"
```

```bash
chmod +x /usr/local/bin/mongodb-backup.sh

# Add to cron (daily at 2 AM)
echo "0 2 * * * /usr/local/bin/mongodb-backup.sh" | sudo tee -a /etc/crontab
```

---

## 11. Monitoring

### Check Status

```bash
# Service status
sudo systemctl status mongod

# Server stats
mongosh --eval "db.serverStatus()"

# Connection stats
mongosh --eval "db.serverStatus().connections"

# Current operations
mongosh --eval "db.currentOp()"
```

### Log Monitoring

```bash
# View logs
sudo tail -f /var/log/mongodb/mongod.log

# Check for errors
sudo grep -i error /var/log/mongodb/mongod.log
```

### Database Stats

```javascript
// In mongosh
use mydb
db.stats()
db.collection.stats()

// Index usage
db.collection.aggregate([{ $indexStats: {} }])
```

---

## 12. Performance Tuning

### System Limits

```bash
sudo nano /etc/security/limits.conf
```

Add:
```
mongodb soft nofile 64000
mongodb hard nofile 64000
mongodb soft nproc 64000
mongodb hard nproc 64000
```

### Disable Transparent Huge Pages

```bash
sudo nano /etc/systemd/system/disable-thp.service
```

```ini
[Unit]
Description=Disable Transparent Huge Pages
After=sysinit.target local-fs.target

[Service]
Type=oneshot
ExecStart=/bin/sh -c 'echo never > /sys/kernel/mm/transparent_hugepage/enabled && echo never > /sys/kernel/mm/transparent_hugepage/defrag'

[Install]
WantedBy=basic.target
```

```bash
sudo systemctl daemon-reload
sudo systemctl enable disable-thp
sudo systemctl start disable-thp
```

### Recommended Indexes

```javascript
// Create indexes for common queries
db.users.createIndex({ email: 1 }, { unique: true })
db.users.createIndex({ createdAt: -1 })
db.orders.createIndex({ userId: 1, createdAt: -1 })

// Check existing indexes
db.collection.getIndexes()

// Drop unused indexes
db.collection.dropIndex("index_name")
```

---

## 13. Replica Set (High Availability)

### Initialize Replica Set

```yaml
# mongod.conf
replication:
  replSetName: "rs0"
```

```bash
sudo systemctl restart mongod
```

```javascript
// In mongosh
rs.initiate({
  _id: "rs0",
  members: [
    { _id: 0, host: "mongo1.example.com:27017" },
    { _id: 1, host: "mongo2.example.com:27017" },
    { _id: 2, host: "mongo3.example.com:27017" }
  ]
})

// Check status
rs.status()
```

---

## 14. Troubleshooting

### MongoDB Won't Start

```bash
# Check logs
sudo tail -100 /var/log/mongodb/mongod.log

# Check permissions
sudo chown -R mongodb:mongodb /var/lib/mongodb
sudo chown -R mongodb:mongodb /var/log/mongodb

# Check disk space
df -h

# Check lock file
sudo rm /var/lib/mongodb/mongod.lock
sudo systemctl start mongod
```

### Connection Refused

```bash
# Check if running
sudo systemctl status mongod

# Check port
sudo ss -tlnp | grep 27017

# Check bind IP in config
grep bindIp /etc/mongod.conf
```

### Authentication Failed

```bash
# Connect without auth and check users
mongosh
use admin
db.getUsers()

# Reset admin password
db.changeUserPassword("admin", "newpassword")
```

---

## Quick Reference

| Task | Command |
|------|---------|
| Start MongoDB | `sudo systemctl start mongod` |
| Stop MongoDB | `sudo systemctl stop mongod` |
| Restart MongoDB | `sudo systemctl restart mongod` |
| Check status | `sudo systemctl status mongod` |
| View logs | `sudo tail -f /var/log/mongodb/mongod.log` |
| Connect shell | `mongosh` |
| Backup | `mongodump --out /backup/` |
| Restore | `mongorestore /backup/` |

---

## Security Checklist

- [ ] Authentication enabled
- [ ] Admin user created with strong password
- [ ] Application users with minimal privileges
- [ ] Bind to localhost or specific IPs only
- [ ] Firewall configured
- [ ] Regular backups scheduled
- [ ] Logs monitored
- [ ] TLS/SSL enabled (for remote connections)

---

## Next Steps

1. **[Node.js Deployment](./deploy-nodejs-app.md)** - Connect your app
2. **[Backup Strategies](./backup-strategies.md)** - Automated backups
3. **[Monitoring](./server-monitoring-guide.md)** - Production monitoring

---

✅ MongoDB is now installed and secured!
