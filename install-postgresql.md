# Install PostgreSQL

Complete guide for installing and configuring PostgreSQL on Ubuntu/Debian.

---

## 1. Install PostgreSQL

```bash
sudo apt update
sudo apt install postgresql postgresql-contrib -y
sudo systemctl start postgresql
sudo systemctl enable postgresql
```

---

## 2. Initial Setup

```bash
# Switch to postgres user
sudo -i -u postgres
psql

# Create user and database
CREATE USER myappuser WITH PASSWORD 'StrongPassword123!';
CREATE DATABASE myapp OWNER myappuser;
GRANT ALL PRIVILEGES ON DATABASE myapp TO myappuser;
\q
```

---

## 3. Authentication (pg_hba.conf)

```bash
sudo nano /etc/postgresql/16/main/pg_hba.conf
```

```
local   all     all                     scram-sha-256
host    all     all     127.0.0.1/32    scram-sha-256
host    all     all     ::1/128         scram-sha-256
```

```bash
sudo systemctl restart postgresql
```

---

## 4. Configuration (postgresql.conf)

```bash
sudo nano /etc/postgresql/16/main/postgresql.conf
```

```ini
listen_addresses = 'localhost'
port = 5432
max_connections = 200
shared_buffers = 256MB
effective_cache_size = 768MB
work_mem = 16MB
```

---

## 5. Connect

```bash
psql -U myappuser -d myapp -h localhost

# Connection string
postgresql://myappuser:password@localhost:5432/myapp
```

---

## 6. Remote Access

```bash
# postgresql.conf
listen_addresses = '*'

# pg_hba.conf
host    all    all    10.0.0.0/24    scram-sha-256

# Firewall
sudo ufw allow from 10.0.0.5 to any port 5432
sudo systemctl restart postgresql
```

---

## 7. Backup & Restore

```bash
# Backup
pg_dump -U postgres myapp > backup.sql
pg_dump -U postgres -Fc myapp > backup.dump

# Restore
psql -U postgres -d myapp < backup.sql
pg_restore -U postgres -d myapp backup.dump
```

---

## 8. Common Commands

```sql
\l              -- List databases
\c dbname       -- Connect to database
\dt             -- List tables
\du             -- List users
\d table_name   -- Describe table

CREATE DATABASE mydb;
DROP DATABASE mydb;
CREATE USER username WITH PASSWORD 'pass';
GRANT ALL PRIVILEGES ON DATABASE mydb TO user;
```

---

## 9. Maintenance

```sql
VACUUM ANALYZE;
REINDEX DATABASE myapp;
```

---

## Quick Reference

| Task | Command |
|------|---------|
| Start | `sudo systemctl start postgresql` |
| Connect | `sudo -u postgres psql` |
| Backup | `pg_dump -U postgres dbname > backup.sql` |
| Logs | `sudo tail -f /var/log/postgresql/*.log` |

---

✅ PostgreSQL is ready!
