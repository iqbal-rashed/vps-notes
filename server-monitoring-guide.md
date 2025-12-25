# Server Monitoring Guide

Essential monitoring tools and commands for VPS management.

---

## 1. System Resources

### CPU & Memory

```bash
# Real-time monitoring
htop

# CPU info
top
lscpu
nproc

# Memory
free -h
cat /proc/meminfo
```

### Disk Usage

```bash
# Disk space
df -h

# Directory sizes
du -sh /var/www/*
du -sh * | sort -rh | head -10

# Disk I/O
iostat
iotop
```

---

## 2. Network Monitoring

```bash
# Open ports
ss -tlnp
netstat -tlnp

# Active connections
ss -tn
netstat -an | grep ESTABLISHED

# Network traffic
iftop
nethogs

# Bandwidth test
speedtest-cli
```

---

## 3. Process Monitoring

```bash
# All processes
ps aux

# Search process
ps aux | grep nginx

# Process tree
pstree

# Kill process
kill PID
kill -9 PID
```

---

## 4. Log Monitoring

```bash
# System logs
sudo journalctl -f
sudo tail -f /var/log/syslog

# Nginx logs
sudo tail -f /var/log/nginx/access.log
sudo tail -f /var/log/nginx/error.log

# Auth logs
sudo tail -f /var/log/auth.log

# Search logs
sudo grep -i error /var/log/syslog
```

---

## 5. Service Status

```bash
# Check service
sudo systemctl status nginx
sudo systemctl status mysql
sudo systemctl status mongodb

# List all services
sudo systemctl list-units --type=service

# Failed services
sudo systemctl --failed
```

---

## 6. Security Monitoring

```bash
# Failed login attempts
sudo grep "Failed password" /var/log/auth.log | tail -20

# Currently logged in users
who
w
last

# Check for rootkits
sudo rkhunter --check

# Open files by process
lsof -i :80
```

---

## 7. Quick Health Check Script

```bash
#!/bin/bash
# /usr/local/bin/health-check.sh

echo "=== System Health Check ==="
echo ""

echo "=== CPU & Memory ==="
uptime
free -h
echo ""

echo "=== Disk Usage ==="
df -h / | tail -1
echo ""

echo "=== Services ==="
for service in nginx mysql mongod redis-server; do
    if systemctl is-active --quiet $service 2>/dev/null; then
        echo "✓ $service is running"
    else
        echo "✗ $service is NOT running"
    fi
done
echo ""

echo "=== Top Processes ==="
ps aux --sort=-%mem | head -6
```

```bash
chmod +x /usr/local/bin/health-check.sh
```

---

## 8. Monitoring Tools

### Install Useful Tools

```bash
sudo apt install -y htop iotop iftop nethogs sysstat
```

### UptimeRobot (External)

- Free external monitoring
- HTTP, ping, port checks
- Email/webhook alerts
- https://uptimerobot.com

---

## Quick Reference

| Task | Command |
|------|---------|
| CPU/Memory | `htop` |
| Disk space | `df -h` |
| Open ports | `ss -tlnp` |
| Logs | `sudo journalctl -f` |
| Services | `sudo systemctl status service` |
| Connections | `ss -tn` |

---

✅ Server monitoring configured!
