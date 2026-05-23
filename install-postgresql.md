# Install PostgreSQL

PostgreSQL (Postgres) is a powerful open-source database. Great for apps that need complex queries or transactions.

---

## Step 1 — Install

```bash
sudo apt update
sudo apt install postgresql postgresql-contrib -y
sudo systemctl start postgresql
sudo systemctl enable postgresql
```

Check it's running:
```bash
sudo systemctl status postgresql
```

---

## Step 2 — Create a Database and User

PostgreSQL uses its own system user called `postgres`. Switch to it:

```bash
sudo -i -u postgres
psql
```

Run these commands inside the psql shell:

```sql
-- Create a user for your app
CREATE USER appuser WITH PASSWORD 'StrongPassword123!';

-- Create a database owned by that user
CREATE DATABASE myapp OWNER appuser;

-- Give the user full access
GRANT ALL PRIVILEGES ON DATABASE myapp TO appuser;

-- Exit
\q
```

Exit back to your normal user:
```bash
exit
```

**Connection string for your app's `.env`:**
```
DATABASE_URL=postgresql://appuser:StrongPassword123!@localhost:5432/myapp
```

---

## Step 3 — Connect to Your Database

```bash
# Connect as your app user
psql -U appuser -d myapp -h localhost

# Connect as postgres (admin)
sudo -u postgres psql
```

---

## Useful psql Commands

```sql
\l              -- List all databases
\c myapp        -- Switch to a database
\dt             -- List tables in current database
\du             -- List all users
\d tablename    -- Describe a table
\q              -- Quit

-- Common SQL
CREATE TABLE users (id SERIAL PRIMARY KEY, name VARCHAR(100), email VARCHAR(255));
SELECT * FROM users;
DROP TABLE users;
```

---

## Allow Only Localhost Connections (Security)

Make sure Postgres only accepts local connections. Check this file:

```bash
sudo nano /etc/postgresql/*/main/postgresql.conf
```

This line should say `localhost`, not `*`:
```ini
listen_addresses = 'localhost'
```

Restart after any config change:
```bash
sudo systemctl restart postgresql
```

---

## Backup and Restore

```bash
# Backup one database
sudo -u postgres pg_dump myapp > /backup/myapp_$(date +%Y%m%d).sql

# Backup all databases
sudo -u postgres pg_dumpall | gzip > /backup/postgres_$(date +%Y%m%d).sql.gz

# Restore
sudo -u postgres psql myapp < /backup/myapp_20260101.sql
```

Auto-backup (daily at 3 AM):
```bash
sudo nano /usr/local/bin/postgres-backup.sh
```
```bash
#!/bin/bash
sudo -u postgres pg_dumpall | gzip > /backup/postgres_$(date +%Y%m%d).sql.gz
find /backup -name "postgres_*.sql.gz" -mtime +7 -delete
```
```bash
chmod +x /usr/local/bin/postgres-backup.sh
echo "0 3 * * * /usr/local/bin/postgres-backup.sh" | sudo tee -a /etc/crontab
```

---

## Troubleshooting

**Connection refused:**
```bash
sudo systemctl status postgresql
sudo tail -20 /var/log/postgresql/*.log
```

**Permission denied:**
```bash
sudo -u postgres psql
GRANT ALL PRIVILEGES ON DATABASE myapp TO appuser;
```

---

## Quick Reference

| Task | Command |
|------|---------|
| Start | `sudo systemctl start postgresql` |
| Connect as admin | `sudo -u postgres psql` |
| Connect as user | `psql -U appuser -d myapp -h localhost` |
| Backup | `sudo -u postgres pg_dump myapp > backup.sql` |
| Logs | `sudo tail -f /var/log/postgresql/*.log` |
