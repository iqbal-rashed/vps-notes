# Install MySQL (or MariaDB)

MySQL and MariaDB are interchangeable for most apps. MariaDB is the default in most Ubuntu setups and is slightly faster.

**Pick one:**
- **MySQL** — Industry standard, used by most tutorials
- **MariaDB** — Drop-in replacement for MySQL, slightly faster, open source

---

## Install MySQL

```bash
sudo apt update
sudo apt install mysql-server -y
sudo systemctl start mysql
sudo systemctl enable mysql
```

## Install MariaDB (Alternative)

```bash
sudo apt update
sudo apt install mariadb-server -y
sudo systemctl start mariadb
sudo systemctl enable mariadb
```

---

## Secure the Installation (Do This Right Away)

```bash
sudo mysql_secure_installation
```

Answer the questions like this:
1. Current root password → press Enter (no password yet)
2. Switch to unix_socket auth → `n`
3. Change root password → `Y` → enter a strong password
4. Remove anonymous users → `Y`
5. Disallow root login remotely → `Y`
6. Remove test database → `Y`
7. Reload privilege tables → `Y`

---

## Create a Database and User for Your App

Connect as root:
```bash
sudo mysql
```

Run these SQL commands (change names and password):
```sql
-- Create a database for your app
CREATE DATABASE myapp CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;

-- Create a user that only has access to this database
CREATE USER 'appuser'@'localhost' IDENTIFIED BY 'StrongPassword123!';
GRANT ALL PRIVILEGES ON myapp.* TO 'appuser'@'localhost';
FLUSH PRIVILEGES;

-- Check it worked
SHOW DATABASES;
exit
```

**Connection string for your app's `.env`:**
```
DATABASE_URL=mysql://appuser:StrongPassword123!@localhost:3306/myapp
```

---

## Useful SQL Commands

```sql
SHOW DATABASES;               -- List all databases
USE myapp;                    -- Switch to a database
SHOW TABLES;                  -- List tables
DESCRIBE users;               -- Show table structure

-- User management
SELECT user, host FROM mysql.user;   -- List all users
DROP USER 'username'@'localhost';    -- Delete a user
ALTER USER 'username'@'localhost' IDENTIFIED BY 'newpassword';  -- Change password
```

---

## Backup and Restore

```bash
# Backup one database
mysqldump -u root -p myapp > /backup/myapp_$(date +%Y%m%d).sql

# Backup all databases
mysqldump -u root -p --all-databases | gzip > /backup/all_$(date +%Y%m%d).sql.gz

# Restore
mysql -u root -p myapp < /backup/myapp_20260101.sql
```

Auto-backup (daily at 3 AM):
```bash
sudo nano /usr/local/bin/mysql-backup.sh
```
```bash
#!/bin/bash
mysqldump -u root -pYOUR_ROOT_PASSWORD --all-databases | \
  gzip > /backup/mysql_$(date +%Y%m%d).sql.gz
find /backup -name "mysql_*.sql.gz" -mtime +7 -delete
```
```bash
chmod +x /usr/local/bin/mysql-backup.sh
echo "0 3 * * * /usr/local/bin/mysql-backup.sh" | sudo tee -a /etc/crontab
```

---

## Troubleshooting

**Can't connect:**
```bash
sudo systemctl status mysql
sudo tail -50 /var/log/mysql/error.log
```

**Forgot root password:**
```bash
sudo systemctl stop mysql
sudo mysqld_safe --skip-grant-tables &
mysql -u root
UPDATE mysql.user SET authentication_string=null WHERE User='root';
FLUSH PRIVILEGES;
ALTER USER 'root'@'localhost' IDENTIFIED BY 'NewPassword!';
sudo systemctl restart mysql
```

---

## Quick Reference

| Task | MySQL | MariaDB |
|------|-------|---------|
| Start | `sudo systemctl start mysql` | `sudo systemctl start mariadb` |
| Connect | `sudo mysql` or `mysql -u root -p` | same |
| Config file | `/etc/mysql/mysql.conf.d/mysqld.cnf` | `/etc/mysql/mariadb.conf.d/50-server.cnf` |
| Error log | `/var/log/mysql/error.log` | same |
