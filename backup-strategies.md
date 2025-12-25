# Backup Strategies

Complete guide for automated backups with retention policies.

---

## 1. File System Backups

### Using rsync

```bash
#!/bin/bash
# /usr/local/bin/backup-files.sh

SOURCE="/var/www"
DEST="/backup/files"
DATE=$(date +%Y%m%d)
RETENTION=7

# Create backup
rsync -avz --delete $SOURCE/ $DEST/$DATE/

# Remove old backups
find $DEST -maxdepth 1 -type d -mtime +$RETENTION -exec rm -rf {} \;

echo "Backup completed: $DATE"
```

### Using tar

```bash
#!/bin/bash
tar -czf /backup/www_$(date +%Y%m%d).tar.gz /var/www

# With exclusions
tar --exclude='node_modules' --exclude='.git' -czf backup.tar.gz /var/www
```

---

## 2. Database Backups

### MySQL/MariaDB

```bash
#!/bin/bash
mysqldump -u root -p'password' --all-databases | gzip > /backup/mysql_$(date +%Y%m%d).sql.gz
```

### PostgreSQL

```bash
#!/bin/bash
sudo -u postgres pg_dumpall | gzip > /backup/postgres_$(date +%Y%m%d).sql.gz
```

### MongoDB

```bash
#!/bin/bash
mongodump --uri="mongodb://user:pass@localhost:27017" --gzip --archive=/backup/mongo_$(date +%Y%m%d).gz
```

---

## 3. Automated Backup Script

```bash
#!/bin/bash
# /usr/local/bin/full-backup.sh

set -e
BACKUP_DIR="/backup"
DATE=$(date +%Y%m%d_%H%M%S)
RETENTION_DAYS=7

mkdir -p $BACKUP_DIR/{files,databases}

# Files backup
echo "Backing up files..."
tar --exclude='node_modules' -czf $BACKUP_DIR/files/www_$DATE.tar.gz /var/www

# Database backups
echo "Backing up databases..."
mysqldump -u root -p'pass' --all-databases | gzip > $BACKUP_DIR/databases/mysql_$DATE.sql.gz

# Cleanup old backups
find $BACKUP_DIR -name "*.gz" -mtime +$RETENTION_DAYS -delete

echo "✅ Backup completed: $DATE"
```

---

## 4. Cron Schedule

```bash
# Edit crontab
sudo crontab -e

# Daily at 3 AM
0 3 * * * /usr/local/bin/full-backup.sh >> /var/log/backup.log 2>&1

# Weekly on Sunday at 2 AM
0 2 * * 0 /usr/local/bin/full-backup.sh >> /var/log/backup.log 2>&1
```

---

## 5. Cloud Backup (S3)

### Install AWS CLI

```bash
sudo apt install awscli -y
aws configure
```

### Sync to S3

```bash
#!/bin/bash
aws s3 sync /backup s3://my-backup-bucket/$(hostname)/ --delete

# Or copy specific file
aws s3 cp /backup/mysql_20241225.sql.gz s3://my-bucket/backups/
```

---

## 6. Remote Backup with rsync

```bash
#!/bin/bash
rsync -avz -e "ssh -i /path/to/key" /backup/ user@backup-server:/backups/$(hostname)/
```

---

## Quick Reference

| Type | Command |
|------|---------|
| Files | `rsync -avz source/ dest/` |
| MySQL | `mysqldump -u root -p db > backup.sql` |
| PostgreSQL | `pg_dump db > backup.sql` |
| MongoDB | `mongodump --out /backup/` |
| Compress | `tar -czf backup.tar.gz /path` |

---

✅ Backups configured!
