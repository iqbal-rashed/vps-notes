# Backup Strategies

Backups protect you when things go wrong — server crashes, accidental deletes, hacks. Set them up once and forget about them.

**Rule of thumb:** Keep 7 days of backups. Store them somewhere other than your server (S3, Backblaze, etc.).

---

## Backup Files (Your Website/App Files)

```bash
sudo nano /usr/local/bin/backup-files.sh
```

```bash
#!/bin/bash
BACKUP_DIR="/backup/files"
DATE=$(date +%Y%m%d)

mkdir -p $BACKUP_DIR

# Backup your web files (skip node_modules and .git)
tar --exclude='*/node_modules' --exclude='*/.git' \
    -czf $BACKUP_DIR/www_$DATE.tar.gz /var/www

# Delete backups older than 7 days
find $BACKUP_DIR -name "www_*.tar.gz" -mtime +7 -delete

echo "File backup done: www_$DATE.tar.gz"
```

```bash
chmod +x /usr/local/bin/backup-files.sh
```

---

## Backup Databases

### MySQL / MariaDB

```bash
sudo nano /usr/local/bin/backup-mysql.sh
```

```bash
#!/bin/bash
BACKUP_DIR="/backup/mysql"
DATE=$(date +%Y%m%d)

mkdir -p $BACKUP_DIR

mysqldump -u root -p'YOUR_ROOT_PASSWORD' --all-databases | \
    gzip > $BACKUP_DIR/mysql_$DATE.sql.gz

find $BACKUP_DIR -name "mysql_*.sql.gz" -mtime +7 -delete

echo "MySQL backup done: mysql_$DATE.sql.gz"
```

### PostgreSQL

```bash
sudo nano /usr/local/bin/backup-postgres.sh
```

```bash
#!/bin/bash
BACKUP_DIR="/backup/postgres"
DATE=$(date +%Y%m%d)

mkdir -p $BACKUP_DIR

sudo -u postgres pg_dumpall | gzip > $BACKUP_DIR/postgres_$DATE.sql.gz

find $BACKUP_DIR -name "postgres_*.sql.gz" -mtime +7 -delete

echo "PostgreSQL backup done: postgres_$DATE.sql.gz"
```

### MongoDB

```bash
sudo nano /usr/local/bin/backup-mongo.sh
```

```bash
#!/bin/bash
BACKUP_DIR="/backup/mongo"
DATE=$(date +%Y%m%d)

mkdir -p $BACKUP_DIR

mongodump --uri="mongodb://admin:PASSWORD@localhost:27017" \
    --gzip --archive=$BACKUP_DIR/mongo_$DATE.gz

find $BACKUP_DIR -name "mongo_*.gz" -mtime +7 -delete

echo "MongoDB backup done: mongo_$DATE.gz"
```

Make all scripts executable:
```bash
chmod +x /usr/local/bin/backup-*.sh
```

---

## Schedule Automatic Backups (Cron)

Run backups every night:

```bash
sudo crontab -e
```

Add these lines:
```
# Files backup at 1 AM
0 1 * * * /usr/local/bin/backup-files.sh >> /var/log/backup.log 2>&1

# Database backup at 2 AM
0 2 * * * /usr/local/bin/backup-mysql.sh >> /var/log/backup.log 2>&1
```

Check the log to make sure it's working:
```bash
cat /var/log/backup.log
```

---

## Upload Backups to the Cloud (Important!)

Backups on the same server don't help if the server dies. Upload them off-server.

### Upload to AWS S3

```bash
# Install AWS CLI
sudo apt install awscli -y
aws configure  # Enter your AWS keys

# Upload a file
aws s3 cp /backup/mysql/mysql_$(date +%Y%m%d).sql.gz s3://your-bucket-name/backups/

# Sync entire backup folder
aws s3 sync /backup/ s3://your-bucket-name/backups/
```

Add to your backup scripts:
```bash
aws s3 sync /backup/ s3://your-bucket-name/backups/ --delete
```

### Upload to Backblaze B2 (Cheaper Than S3)

```bash
pip install b2
b2 authorize-account YOUR_APP_KEY_ID YOUR_APP_KEY
b2 sync /backup/ b2://your-bucket-name/backups/
```

---

## Restore From Backup

### MySQL
```bash
# Restore all databases
gunzip < /backup/mysql/mysql_20260101.sql.gz | mysql -u root -p
```

### PostgreSQL
```bash
gunzip < /backup/postgres/postgres_20260101.sql.gz | sudo -u postgres psql
```

### MongoDB
```bash
mongorestore --uri="mongodb://admin:PASSWORD@localhost:27017" \
    --gzip --archive=/backup/mongo/mongo_20260101.gz
```

### Files
```bash
tar -xzf /backup/files/www_20260101.tar.gz -C /
```

---

## Check Backup Status

```bash
# See all backups and their sizes
ls -lh /backup/mysql/
ls -lh /backup/files/

# See last backup log
tail -20 /var/log/backup.log
```

---

## Quick Checklist

- [ ] File backup script created and tested
- [ ] Database backup script created and tested
- [ ] Backups scheduled with cron
- [ ] Backups uploading to cloud storage
- [ ] Tested restoring from a backup
- [ ] Checked that old backups are being deleted (not filling disk)
