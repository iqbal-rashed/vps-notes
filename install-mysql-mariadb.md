# Install MySQL/MariaDB

Complete guide for installing and securing MySQL or MariaDB on Ubuntu/Debian servers.

---

## MySQL vs MariaDB

| Feature | MySQL | MariaDB |
|---------|-------|---------|
| Default in Ubuntu | No | Yes (some versions) |
| Performance | Good | Slightly better |
| GPL Licensed | Dual | Pure GPL |
| Drop-in replacement | - | Yes |

---

## Install MySQL

### MySQL 8.0

```bash
# Update packages
sudo apt update

# Install MySQL
sudo apt install mysql-server -y

# Start and enable
sudo systemctl start mysql
sudo systemctl enable mysql

# Check status
sudo systemctl status mysql
```

### Verify Installation

```bash
mysql --version
```

---

## Install MariaDB

### MariaDB 10.11 LTS

```bash
# Update packages
sudo apt update

# Install MariaDB
sudo apt install mariadb-server mariadb-client -y

# Start and enable
sudo systemctl start mariadb
sudo systemctl enable mariadb

# Check status
sudo systemctl status mariadb
```

---

## Secure Installation (Critical!)

Run the security script:

```bash
sudo mysql_secure_installation
```

Answer the prompts:

1. **Enter current root password**: Press Enter (no password yet)
2. **Switch to unix_socket authentication**: `n` (optional)
3. **Change root password**: `Y` and enter new password
4. **Remove anonymous users**: `Y`
5. **Disallow root login remotely**: `Y`
6. **Remove test database**: `Y`
7. **Reload privilege tables**: `Y`

---

## Initial Configuration

### Connect as Root

```bash
# MySQL 8.0+
sudo mysql

# Or with password
mysql -u root -p
```

### Create Admin User

```sql
-- Create new admin user
CREATE USER 'admin'@'localhost' IDENTIFIED BY 'StrongPassword123!';
GRANT ALL PRIVILEGES ON *.* TO 'admin'@'localhost' WITH GRANT OPTION;
FLUSH PRIVILEGES;
```

### Create Application Database and User

```sql
-- Create database
CREATE DATABASE myapp CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;

-- Create application user
CREATE USER 'appuser'@'localhost' IDENTIFIED BY 'AppPassword123!';
GRANT ALL PRIVILEGES ON myapp.* TO 'appuser'@'localhost';
FLUSH PRIVILEGES;

-- Verify
SHOW DATABASES;
SELECT user, host FROM mysql.user;
```

---

## Configuration File

### MySQL Configuration

```bash
sudo nano /etc/mysql/mysql.conf.d/mysqld.cnf
```

### MariaDB Configuration

```bash
sudo nano /etc/mysql/mariadb.conf.d/50-server.cnf
```

### Recommended Settings

```ini
[mysqld]
# Basic Settings
user            = mysql
pid-file        = /var/run/mysqld/mysqld.pid
socket          = /var/run/mysqld/mysqld.sock
port            = 3306
basedir         = /usr
datadir         = /var/lib/mysql
tmpdir          = /tmp

# Bind address
bind-address    = 127.0.0.1    # Localhost only
# bind-address  = 0.0.0.0      # All interfaces (requires firewall!)

# Character set
character-set-server  = utf8mb4
collation-server      = utf8mb4_unicode_ci

# InnoDB Settings (adjust based on RAM)
innodb_buffer_pool_size = 1G           # 50-70% of RAM
innodb_log_file_size    = 256M
innodb_flush_log_at_trx_commit = 1
innodb_file_per_table   = 1

# Connection Settings
max_connections         = 200
max_allowed_packet      = 64M
wait_timeout            = 600
interactive_timeout     = 600

# Query Cache (MySQL 5.7, deprecated in 8.0)
# query_cache_type      = 1
# query_cache_size      = 64M

# Slow Query Log
slow_query_log          = 1
slow_query_log_file     = /var/log/mysql/slow.log
long_query_time         = 2

# Error Log
log_error               = /var/log/mysql/error.log

# Binary Log (for replication)
# log_bin               = /var/log/mysql/mysql-bin.log
# expire_logs_days      = 10
# max_binlog_size       = 100M
```

### Apply Changes

```bash
sudo systemctl restart mysql
# or
sudo systemctl restart mariadb
```

---

## Allow Remote Connections

> ⚠️ **Warning:** Only enable if absolutely needed and use firewall!

### Update Configuration

```ini
# In mysqld.cnf
bind-address = 0.0.0.0
```

### Create Remote User

```sql
-- Create user that can connect from specific IP
CREATE USER 'remoteuser'@'10.0.0.5' IDENTIFIED BY 'RemotePassword123!';
GRANT ALL PRIVILEGES ON myapp.* TO 'remoteuser'@'10.0.0.5';
FLUSH PRIVILEGES;

-- Or from any IP (less secure)
CREATE USER 'remoteuser'@'%' IDENTIFIED BY 'RemotePassword123!';
GRANT ALL PRIVILEGES ON myapp.* TO 'remoteuser'@'%';
FLUSH PRIVILEGES;
```

### Firewall Rules

```bash
# Only allow from specific IP
sudo ufw allow from 10.0.0.5 to any port 3306
```

### Restart Service

```bash
sudo systemctl restart mysql
```

### Connection String

```
mysql://remoteuser:password@server-ip:3306/myapp
```

---

## Backup and Restore

### Create Backup

```bash
# Single database
mysqldump -u root -p myapp > /backup/myapp_$(date +%Y%m%d).sql

# All databases
mysqldump -u root -p --all-databases > /backup/all_databases_$(date +%Y%m%d).sql

# Compressed backup
mysqldump -u root -p myapp | gzip > /backup/myapp_$(date +%Y%m%d).sql.gz

# With routines and triggers
mysqldump -u root -p --routines --triggers myapp > /backup/myapp_full.sql
```

### Restore Backup

```bash
# Restore single database
mysql -u root -p myapp < /backup/myapp_20241225.sql

# Restore from compressed
gunzip < /backup/myapp_20241225.sql.gz | mysql -u root -p myapp

# Restore all databases
mysql -u root -p < /backup/all_databases.sql
```

### Automated Backup Script

```bash
#!/bin/bash
# /usr/local/bin/mysql-backup.sh

BACKUP_DIR="/backup/mysql"
DATE=$(date +%Y%m%d_%H%M%S)
RETENTION_DAYS=7
MYSQL_USER="backup_user"
MYSQL_PASSWORD="BackupPassword123!"

# Create backup directory
mkdir -p $BACKUP_DIR

# Backup all databases
mysqldump -u $MYSQL_USER -p$MYSQL_PASSWORD --all-databases --routines --triggers | gzip > $BACKUP_DIR/all_databases_$DATE.sql.gz

# Remove old backups
find $BACKUP_DIR -name "*.sql.gz" -mtime +$RETENTION_DAYS -delete

echo "Backup completed: all_databases_$DATE.sql.gz"
```

```bash
chmod +x /usr/local/bin/mysql-backup.sh

# Add to cron (daily at 3 AM)
echo "0 3 * * * /usr/local/bin/mysql-backup.sh" | sudo tee -a /etc/crontab
```

---

## Common Operations

### Database Operations

```sql
-- Show all databases
SHOW DATABASES;

-- Create database
CREATE DATABASE mydb CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;

-- Use database
USE mydb;

-- Drop database
DROP DATABASE mydb;

-- Show tables
SHOW TABLES;

-- Show table structure
DESCRIBE table_name;
```

### User Management

```sql
-- Show all users
SELECT user, host FROM mysql.user;

-- Create user
CREATE USER 'username'@'localhost' IDENTIFIED BY 'password';

-- Grant privileges
GRANT ALL PRIVILEGES ON database.* TO 'username'@'localhost';
GRANT SELECT, INSERT, UPDATE ON database.* TO 'username'@'localhost';

-- Revoke privileges
REVOKE ALL PRIVILEGES ON database.* FROM 'username'@'localhost';

-- Change password
ALTER USER 'username'@'localhost' IDENTIFIED BY 'newpassword';

-- Delete user
DROP USER 'username'@'localhost';

-- Apply changes
FLUSH PRIVILEGES;
```

### Performance Queries

```sql
-- Show running queries
SHOW PROCESSLIST;

-- Kill a query
KILL <process_id>;

-- Show status
SHOW STATUS;
SHOW STATUS LIKE 'Connections';
SHOW STATUS LIKE 'Threads%';

-- Show variables
SHOW VARIABLES LIKE 'max_connections';
SHOW VARIABLES LIKE 'innodb%';

-- Table sizes
SELECT 
    table_schema AS 'Database',
    ROUND(SUM(data_length + index_length) / 1024 / 1024, 2) AS 'Size (MB)'
FROM information_schema.tables
GROUP BY table_schema;
```

---

## Performance Tuning

### Check Current Settings

```bash
# Install mysqltuner
sudo apt install mysqltuner -y

# Run tuner
sudo mysqltuner
```

### Key Settings to Tune

```ini
# Memory (50-70% of available RAM)
innodb_buffer_pool_size = 2G

# Connections (based on app needs)
max_connections = 300

# Thread cache
thread_cache_size = 50

# Table cache
table_open_cache = 4000

# Query cache (MySQL 5.7 only)
query_cache_type = 1
query_cache_size = 128M
query_cache_limit = 2M
```

---

## Troubleshooting

### Can't Connect

```bash
# Check if running
sudo systemctl status mysql

# Check port
sudo ss -tlnp | grep 3306

# Check bind address
grep bind-address /etc/mysql/mysql.conf.d/mysqld.cnf

# Check error log
sudo tail -100 /var/log/mysql/error.log
```

### Access Denied

```bash
# Reset root password
sudo systemctl stop mysql
sudo mysqld_safe --skip-grant-tables &
mysql -u root

UPDATE mysql.user SET authentication_string=null WHERE User='root';
FLUSH PRIVILEGES;
ALTER USER 'root'@'localhost' IDENTIFIED BY 'NewPassword123!';

sudo systemctl restart mysql
```

### Database Corrupted

```bash
# Check tables
mysqlcheck -u root -p --all-databases

# Repair tables
mysqlcheck -u root -p --repair --all-databases
```

### Slow Queries

```sql
-- Enable slow query log
SET GLOBAL slow_query_log = 'ON';
SET GLOBAL long_query_time = 2;

-- Check slow queries
SELECT * FROM mysql.slow_log ORDER BY start_time DESC LIMIT 10;
```

---

## Quick Reference

| Task | Command |
|------|---------|
| Start MySQL | `sudo systemctl start mysql` |
| Stop MySQL | `sudo systemctl stop mysql` |
| Restart MySQL | `sudo systemctl restart mysql` |
| Status | `sudo systemctl status mysql` |
| Connect | `mysql -u root -p` |
| Backup | `mysqldump -u root -p dbname > backup.sql` |
| Restore | `mysql -u root -p dbname < backup.sql` |
| Error log | `sudo tail -f /var/log/mysql/error.log` |

---

## Next Steps

1. **[Application Deployment](./deploy-nodejs-app.md)** - Connect your app
2. **[Backup Strategies](./backup-strategies.md)** - Automated backups
3. **[Monitoring](./server-monitoring-guide.md)** - Database monitoring

---

✅ MySQL/MariaDB is installed and secured!
