# Install MongoDB

Guide for installing MongoDB on Ubuntu and securing it for production.

---

## Step 1 — Install MongoDB

```bash
# Add MongoDB's signing key
curl -fsSL https://www.mongodb.org/static/pgp/server-8.0.asc | \
  sudo gpg -o /usr/share/keyrings/mongodb-server-8.0.gpg --dearmor

# Add MongoDB repository (Ubuntu 22.04)
echo "deb [ arch=amd64,arm64 signed-by=/usr/share/keyrings/mongodb-server-8.0.gpg ] https://repo.mongodb.org/apt/ubuntu jammy/mongodb-org/8.0 multiverse" | \
  sudo tee /etc/apt/sources.list.d/mongodb-org-8.0.list

# Install
sudo apt update
sudo apt install mongodb-org -y
```

For **Ubuntu 24.04**, change `jammy` to `noble` in the repository line.

---

## Step 2 — Start MongoDB

```bash
sudo systemctl start mongod
sudo systemctl enable mongod
sudo systemctl status mongod
```

---

## Step 3 — Create Users and Enable Authentication

By default, MongoDB has no password. Anyone on the server can access it. Fix this now.

```bash
mongosh
```

```javascript
// Step 1: Create an admin user
use admin

db.createUser({
  user: "admin",
  pwd: passwordPrompt(),
  roles: ["userAdminAnyDatabase", "readWriteAnyDatabase"]
})

// Step 2: Create a user just for your app (safer)
use myapp

db.createUser({
  user: "appuser",
  pwd: passwordPrompt(),
  roles: [{ role: "readWrite", db: "myapp" }]
})

exit
```

---

## Step 4 — Turn On Authentication

```bash
sudo nano /etc/mongod.conf
```

Find the `security:` section and set:
```yaml
security:
  authorization: enabled
```

Also make sure MongoDB only listens on localhost:
```yaml
net:
  port: 27017
  bindIp: 127.0.0.1
```

Restart MongoDB:
```bash
sudo systemctl restart mongod
```

---

## Step 5 — Connect With Authentication

```bash
# Connect as admin
mongosh -u admin -p --authenticationDatabase admin

# Connect as app user
mongosh "mongodb://appuser:YOUR_PASSWORD@localhost:27017/myapp"
```

**Connection string for your app's `.env`:**
```
DATABASE_URL=mongodb://appuser:YOUR_PASSWORD@localhost:27017/myapp
```

---

## Basic MongoDB Commands

```javascript
// Show all databases
show dbs

// Switch to a database (creates it if it doesn't exist)
use myapp

// Show collections (like tables)
show collections

// Insert a document
db.users.insertOne({ name: "Alice", email: "alice@example.com" })

// Find all documents
db.users.find()

// Find with a filter
db.users.find({ name: "Alice" })

// Count documents
db.users.countDocuments()

// Delete a document
db.users.deleteOne({ name: "Alice" })
```

---

## Backup and Restore

```bash
# Backup your database
mongodump --uri="mongodb://admin:PASSWORD@localhost:27017" --gzip --archive=/backup/mongo_$(date +%Y%m%d).gz

# Restore from backup
mongorestore --uri="mongodb://admin:PASSWORD@localhost:27017" --gzip --archive=/backup/mongo_20260101.gz
```

Auto-backup script (runs daily at 2 AM):
```bash
sudo nano /usr/local/bin/mongo-backup.sh
```
```bash
#!/bin/bash
mongodump --uri="mongodb://admin:PASSWORD@localhost:27017" \
  --gzip --archive="/backup/mongo_$(date +%Y%m%d).gz"
# Delete backups older than 7 days
find /backup -name "mongo_*.gz" -mtime +7 -delete
```
```bash
chmod +x /usr/local/bin/mongo-backup.sh
echo "0 2 * * * /usr/local/bin/mongo-backup.sh" | sudo tee -a /etc/crontab
```

---

## Troubleshooting

**MongoDB won't start:**
```bash
sudo tail -50 /var/log/mongodb/mongod.log
sudo chown -R mongodb:mongodb /var/lib/mongodb
```

**Authentication failed:**
```bash
# Connect without auth to reset (temporarily disable auth first)
sudo nano /etc/mongod.conf  # comment out authorization: enabled
sudo systemctl restart mongod
mongosh
use admin
db.changeUserPassword("admin", "newpassword")
```

---

## Quick Reference

| Task | Command |
|------|---------|
| Start | `sudo systemctl start mongod` |
| Stop | `sudo systemctl stop mongod` |
| Restart | `sudo systemctl restart mongod` |
| Shell | `mongosh -u admin -p --authenticationDatabase admin` |
| Logs | `sudo tail -f /var/log/mongodb/mongod.log` |
